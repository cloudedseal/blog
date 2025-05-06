---
date: '2025-05-05T17:45:55+08:00'
draft: false
title: 'Syscall Futex'
tags: ["syscall","futex"]
---

**Futex (Fast Userspace Mutex)**

A **futex** is a Linux kernel system call that provides a fast and efficient mechanism for implementing `user-space synchronization` primitives, such as mutexes, semaphores, and condition variables. It `minimizes` kernel involvement `in uncontended cases`, reducing overhead by handling synchronization in userspace when possible.

> Futexes split synchronization into **user-space atomics** (fast) and **kernel-assisted blocking** (slow), optimizing for the common case. The boundary is enforced by hardware (CPU modes) and the need for kernel-managed resources (scheduling, interrupts).


## **How Futex Works?**

1. 没有竞争，CAS 后直接获取锁，不涉及到 syscall
2. 有竞争, futex_wait 让线程 sleep, 等待其他线程 futex_wake 唤醒

大多数时候是没有竞争的，所以就减少了内核的干预。

### **futex 设计原则**

#### **Avoid Kernel Calls When Possible**  
   - Use `atomic operations` for `uncontended cases` (fast path).  
   - Only call the kernel when blocking is required (slow path).

#### **Kernel as a Backstop**  
   - The kernel handles complex tasks like:
     - Managing wait queues.
     - Guaranteeing fairness/priority in thread wakeup.
     - Handling signals/interrupts during blocking.

#### **Spurious Wakeups**  
   - The kernel may wake threads even if the futex value hasn’t changed (e.g., due to `signals`).  
   - **User-space must `recheck` the futex value** after wakeup (e.g., loop in `lock()`).



### **为啥需要减少内核的干预?**

Kernel calls (system calls) are **expensive** compared to user-space operations. 

#### **Context Switch Overhead**  
   - When a thread enters the kernel (e.g., via `syscall`), the CPU must:
     - Save user-space registers. 保存用户空间寄存器
     - Switch to kernel mode (privileged CPU state). CPU 模式切换到特权模式
     - Run kernel code (e.g., scheduler, wait queues). 执行内核代码，比如调度器
     - Restore user-space state afterward. 恢复用户空间寄存器
   - This process takes **hundreds to thousands of CPU cycles**, adding latency.

#### **Scalability**  
   - Frequent kernel calls create contention in the `kernel’s internal data structures` (e.g., locks for process/thread management). 
   - Kernel resources are shared `across all processes`, so overuse hurts overall system performance. 所有进程共享内核资源

#### **Fast-Path Optimization**  
   - Most synchronization operations (e.g., locking a mutex) are **uncontended** (no other thread holds the lock).
   - Handling these cases purely in user-space avoids kernel interaction entirely.


#### **User Space and Kernel 边界**

The boundary is defined by **CPU privilege levels** and the need for kernel-managed resources:

| **User Space**                          | **Kernel Space**                          |
|-----------------------------------------|-------------------------------------------|
| Runs in unprivileged CPU mode (ring 3). | Runs in privileged CPU mode (ring 0).     |
| Directly manipulates user memory.       | Manages hardware, interrupts, scheduling. |
| Uses atomic CPU instructions (e.g., `cmpxchg`). | Uses system calls (e.g., `futex`, `sched_yield`). |
| Handles "fast path" (uncontended case). | Manages "slow path" (blocking/waking threads). |


## **Futex: Balancing User/Kernel Responsibilities**

`futex` 横跨 user/kernel 两个空间

### **User-Space Fast Path (No Contention)**  

- **Atomic Compare-and-Swap (CAS):**  
  ```c
  atomic_compare_exchange_strong(&futex, 0, 1); // Pure user-space
  ```
  - If the lock is free (`futex == 0`), the thread acquires it **without involving the kernel**.  
  - This is a single CPU instruction (e.g., `lock cmpxchg` on x86), blazing fast.

### **Kernel Slow Path (Contention)**  

#### **Blocking with `FUTEX_WAIT`**  
  ```c
  syscall(SYS_futex, &futex, FUTEX_WAIT, 1, ...);
  ```
  - If the lock is held (`futex == 1`), the thread asks the kernel to **block it** until the lock is freed.  
  - The kernel adds the thread to a `wait queue` and `schedules other threads`.

#### **Waking with `FUTEX_WAKE`**  
  ```c
  syscall(SYS_futex, &futex, FUTEX_WAKE, 1, ...);
  ```
  - `On unlock`, the kernel `wakes` one blocked thread.  
  - This involves `scheduler` logic (kernel responsibility).

### **Futex 主要操作**

- **`FUTEX_WAIT`**: Puts the calling thread to `sleep` if the futex word matches the expected value.
```
  If *uaddr == val, the thread goes to sleep (blocks).
  If *uaddr != val, the call returns immediately with EAGAIN.
  If a timeout is set, the thread may also wake up due to timeout (ETIMEDOUT).
```
- **`FUTEX_WAKE`**: Wakes up a specified number of threads waiting on the futex.
- **`FUTEX_WAIT_BITSET` / `FUTEX_WAKE_BITSET`**: Advanced operations for conditional waits using bitmasks (e.g., for condition variables).

### **工作流程示例**

1. **Thread A acquires the lock:**  
   - CAS succeeds in user space (`futex` becomes 1).  
   - No kernel interaction.

2. **Thread B tries to acquire the lock:**  
   - CAS fails (futex is 1).  
   - Calls `FUTEX_WAIT` to block (kernel involvement).

3. **Thread A releases the lock:**  
   - Sets `futex` to 0 (user space).  
   - Calls `FUTEX_WAKE` to unblock Thread B (kernel).


### **Why This Matters in Practice?**

- **Performance:** A futex-based mutex can be **10–100x faster** than a traditional kernel-only mutex under low contention.  
- **Scalability:** Reduces kernel lock contention in highly parallel workloads (e.g., databases, game engines).  
- **Flexibility:** User-space can implement custom synchronization logic (e.g., adaptive mutexes, read/write locks).



1. **Core Concept**:
   - A **futex word** (a 32-bit integer in shared memory) acts as the synchronization variable. 和平台无关, 都是 32bit
   - Threads use atomic operations (e.g., compare-and-swap) to manipulate this word in userspace.
   - Kernel intervention occurs **only during contention** (e.g., when a thread must wait).

2. **Acquiring a Lock (`Uncontended Case`)**:
   - A thread attempts to atomically set the futex word from `0` (unlocked) to `1` (locked).
   - If successful, the lock is acquired without a system call.

3. **Acquiring a Lock (`Contended Case`)**:
   - If the lock is already held (`futex_word == 1`), the thread calls `futex_wait(&futex_word, 1)` to block.
   - The kernel checks if the futex word is still `1` and puts the thread to sleep, avoiding race conditions.

4. **Releasing a Lock**:
   - The thread sets the futex word back to `0` atomically.
   - It then calls `futex_wake(&futex_word, 1)` to wake up one waiting thread.

5. **Handling Spurious Wakeups**:
   - After waking, threads **re-check** the futex word to ensure the lock is available before proceeding.



### **例子理解**

```c {fileName="futex_demo.c"}
#include <linux/futex.h>
#include <sys/syscall.h>
#include <unistd.h>
#include <stdio.h>
#include <pthread.h>

// Futex word (shared between threads)
int futex_var = 0; // 0 = unlocked, 1 = locked

// Wrapper for futex_wait syscall
void futex_wait(int* uaddr, int val) {
    syscall(SYS_futex, uaddr, FUTEX_WAIT, val, NULL, NULL, 0);
}


// Wrapper for futex_wake syscall
void futex_wake(int* uaddr) {
    syscall(SYS_futex, uaddr, FUTEX_WAKE, 1, NULL, NULL, 0);
}

// Thread function with custom sleep time
void* thread_func(void* arg) {
    int sleep_time = *(int*)arg; // Cast void* to int*

    // Try to acquire lock (using GCC atomic compare-and-swap)
    int expected = 0;
    while (!__sync_bool_compare_and_swap(&futex_var, 0, 1)) {
        // Contended case: wait for wakeups
        printf("Thread %lu acquired the lock failed\n", pthread_self());
        futex_wait(&futex_var, 1);

    }

    // Critical section (simulate work with custom sleep)
    printf("Thread %lu acquired the lock success, sleeping for %d seconds\n", 
           pthread_self(), sleep_time);
    sleep(sleep_time); // Customizable sleep duration

    // Release lock
    futex_var = 0;
    futex_wake(&futex_var);

    return NULL;
}


int main() {
    // Define sleep durations for each thread
    int sleep1 = 5;  // First thread sleeps for 1 second
    int sleep2 = 3;  // Second thread sleeps for 3 seconds
    int sleep3 = 2;

    // Create threads
    pthread_t t1, t2, t3;
    printf("%p\n",&futex_var);
    pthread_create(&t1, NULL, thread_func, &sleep1);
    pthread_create(&t2, NULL, thread_func, &sleep2);
    pthread_create(&t3, NULL, thread_func, &sleep3);

    // Wait for threads to finish
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    pthread_join(t3, NULL);

    return 0;
}
```

> gcc -g -pthread -std=gnu11 futex_demo.c -o futex_demo


## **什么是 Spurious Wakeup?**

A **spurious wakeup** occurs when a thread waiting on a synchronization primitive (e.g., a condition variable, futex, or semaphore) **wakes up without being explicitly signaled or broadcasted**. This means the thread resumes execution even though no other thread modified the condition it was waiting for.

Spurious wakeups are **not errors** in the system.
they are a design trade-off in low-level synchronization mechanisms like futexes to **avoid unnecessary overhead** in kernel-space implementations.



### **为啥需要 spurious Wakeups?**

spurious wakeups are allowed for **performance and implementation efficiency**, particularly in systems like Linux futexes. 

为了性能和高效的实现需要 spurious wakeups

#### 1. **Kernel Optimization**
   - The Linux kernel uses shared data structures (e.g., wait queues) for threads waiting on a futex. If multiple threads are waiting on the same futex word, the kernel may wake up **more than one thread** (even if only one is needed) to reduce contention and latency.
   - This avoids the overhead of tracking exactly which thread should wake up.

#### 2. **Signal Handling**
   - A thread waiting on a futex may be interrupted by a **signal** (e.g., `SIGINT`, `SIGALRM`). The kernel wakes up the thread to handle the signal, even if the futex condition hasn’t changed.

#### 3. **Hardware/Architecture Constraints**
   - Some architectures or hardware may not guarantee atomicity between checking the condition and sleeping, leading to race conditions that require re-checking after waking.



## **什么情况会发生 Spurious Wakeups?**


| Scenario                          | Description                                                                 |
|-----------------------------------|-----------------------------------------------------------------------------|
| **Multiple Waiters**              | When many threads wait on the same futex, the kernel may wake multiple threads (e.g., via `FUTEX_WAKE`) even if only one is needed.<br/> 多个等待同一个 futex 的线程, 被 os 一并唤醒, 但最终只有一个获取 futex |
| **Signal Interruption**          | A thread is interrupted by a signal (e.g., Ctrl+C), causing it to wake up prematurely.<br> 虽然进程 sleep 了, 但是仍可以被 os 唤醒处理信号|
| **Timeout Expiration**          | If a timeout is set (e.g., `FUTEX_WAIT_BITSET` with a timeout), the thread may wake up due to the timeout, even if the condition hasn’t changed.<br/> 设置了超时, 超时后可以被 os 唤醒|
| **Kernel Preemption**             | In high-load scenarios, the kernel may preempt a waiting thread for scheduling reasons. <br/>  被 os 抢占调度唤醒|



### **如何处理 Spurious Wakeups?**

To handle spurious wakeups correctly, **always re-check the condition** after waking up. This is a fundamental rule in concurrent programming.

#### **例子1 重新检查条件**

```c
// Shared futex word (0 = unlocked, 1 = locked)
int futex_var = 0;

// Thread attempts to acquire the lock
while (!__sync_bool_compare_and_swap(&futex_var, 0, 1)) {
    // Wait for wakeups if futex_var is still 1
    futex_wait(&futex_var, 1);
}
```

Here’s what happens:
1. If `futex_wait` returns due to a spurious wakeup, the thread **re-checks the condition** (`futex_var == 1`) in the loop.
2. If the condition is still true, the thread calls `futex_wait` again.
3. If the condition is now false (e.g., another thread released the lock), the thread proceeds.

#### **例子2 重新检查条件**

```c
pthread_mutex_lock(&mutex);
while (!condition_met) {
    pthread_cond_wait(&cond, &mutex); // Releases mutex, waits for signal
}
// Re-check condition here (spurious wakeups handled by the loop)
pthread_mutex_unlock(&mutex);
```

The `while` loop ensures that spurious wakeups are harmless.



### **Why Not Prevent Spurious Wakeups?**

Preventing spurious wakeups would require **strict guarantees** from the kernel or library, which could introduce significant overhead. For example:
- Tracking exactly which thread should wake up (e.g., via a queue).
- Adding memory barriers or locks to ensure atomicity between checking and sleeping.

By allowing spurious wakeups, systems like futexes remain lightweight and scalable for high-performance applications.

