---
date: '2025-05-06T15:11:27+08:00'
draft: false
title: 'Java Thread'
---

## java 的线程由 os scheduler 负责调度

Java thread scheduling is **primarily managed by the operating system (OS) scheduler**, but the Java Virtual Machine (JVM) and the application code can `influence` thread behavior in several ways. 

- java 归根结底一个用户空间的程序而已。
- 实际的线程调度根本上还是由操作系统的调度器负责。
- java 只是可以调整相应参数来影响线程的调度。
- 线程最终如何调度 java 无法决定。



### **OS Scheduler Dependency**
- **Native Threads**: Modern JVMs use **native threads** (`one-to-one` mapping with OS threads). The OS scheduler is responsible for allocating CPU time to these threads based on its own algorithms (e.g., time-sharing, priority-based scheduling).
- **Thread Lifecycle**: The OS handles thread `creation`, `scheduling`, `preemption`, and `termination`. Even though Java provides abstractions like `Thread` and `Runnable`, the underlying OS controls execution.


### **java 可以影响 Thread 行为**
While the **`OS scheduler`** has `ultimate authority`, Java applications can **influence thread behavior** through:

#### **1. Thread Priority**

> 设置线程的优先级

- Java allows setting thread priorities (`Thread.setPriority(int priority)`), which are mapped to OS-specific priorities.
  - **Effect**: Higher-priority threads may get more CPU time, but this is **not guaranteed**. OS policies (e.g., Linux's Completely Fair Scheduler) may ignore or reinterpret Java priorities.
  - **Example**: A thread with `Thread.MAX_PRIORITY` might run more frequently than one with `Thread.MIN_PRIORITY`, but the OS decides.

#### **2. Thread Yielding and Sleeping**

> 放弃 CPU 的控制权/休眠

- `Thread.yield()`: Hints to the OS scheduler that the current thread is willing to yield its CPU time. The scheduler may choose another thread of the same priority to run.
- `Thread.sleep(long millis)`: Pauses the thread for a specified time, allowing other threads to execute. This indirectly affects scheduling by reducing contention.

#### **3. Synchronization and Blocking**
- Synchronized blocks, locks (`synchronized`, `ReentrantLock`), and I/O operations (e.g., file/network I/O) can cause threads to block, triggering context switches managed by the OS.
- Contention for shared resources (e.g., locks) can lead to thread starvation or deadlocks, which the application must manage.

#### **4. Thread States and Lifecycle**
- Application code controls thread states (e.g., `start()`, `join()`, `interrupt()`). For example:
  - `start()`: start the thread
  - `join()`: Causes one thread to wait for another's completion.
  - `interrupt()`: Signals a thread to stop, which the application must handle (e.g., via `InterruptedException`).

#### **5. Executor Framework and Thread Pools**
- High-level concurrency utilities (`java.util.concurrent`) abstract thread management. 
- Applications can configure thread pools (e.g., `ThreadPoolExecutor`) to control how tasks are scheduled, but the actual execution still depends on the OS scheduler.


#### **java 层面可以影响 os 调度器的参数总结**
| **Aspect**               | **Controlled By**                  | **Influenced By java**        |
|--------------------------|-------------------------------------|---------------------------------------|
| Scheduling Algorithm     | OS (e.g., Linux CFS, Windows UMS)   | ❌                                    |
| Thread Priority          | OS (mapped from Java priorities)    | ✅ (`setPriority()`)                  |
| Yielding/Sleeping        | OS (honors hints)                   | ✅ (`yield()`, `sleep()`)             |
| Blocking/Synchronization | OS (context switches)               | ✅ (`synchronized`, I/O operations)   |
| Thread Creation/Lifecycle| OS (via JVM)                        | ✅ (`start()`, `join()`, `interrupt()`)|

In short, **the OS scheduler ultimately controls thread execution**, but application code can `influence behavior` through Java's threading APIs, synchronization, and resource management. For fine-grained control, low-level OS tools or real-time systems are required.



### **JVM and OS Interactions**
- The JVM acts as an `intermediary` between Java code and the OS. It maps Java thread operations to OS-specific primitives (e.g., POSIX threads on Unix-like systems).
- Some JVM implementations may optimize thread scheduling (e.g., biased locking for synchronization), but these are `OS-agnostic` and still subject to OS-level scheduling. 说白了, jvm 再怎样优化终究是 userspace 代码, 内核不关心。


### **Limitations of Application Control**
- **No Direct Control**: Java does not expose low-level OS scheduling policies (e.g., `SCHED_FIFO` or `SCHED_RR` in Linux). Applications cannot directly set real-time scheduling policies.
- **Platform Variability**: Thread behavior may differ across OSes (e.g., Windows vs. Linux) or JVM implementations (e.g., HotSpot vs. OpenJ9).



### **Real-Time Java (RTSJ)**
- For applications requiring deterministic scheduling (e.g., real-time systems), the **Real-Time Specification for Java (RTSJ)** provides stricter control over threads, but this requires a real-time JVM and OS support.



---


## AQS 和 OS 关系
The **AbstractQueuedSynchronizer (AQS)** is a foundational class in Java's `java.util.concurrent` package that provides a framework for building **synchronizers** like locks (`ReentrantLock`), barriers (`CyclicBarrier`), latches (`CountDownLatch`), and semaphores (`Semaphore`). 
While the **OS scheduler** still governs thread execution, AQS introduces mechanisms to **manage thread blocking/waking** and **fairness policies**, which directly `influence` thread behavior and scheduling.



### **How AQS Works**
AQS manages synchronization by:
1. **State Management**:  
   - Uses a `volatile int state` to represent the synchronization state (e.g., lock count for `ReentrantLock`, remaining permits for `Semaphore`, or count for `CountDownLatch`).
   - Threads attempt to modify this state atomically (via CAS operations like `compareAndSetState`).

2. **Wait Queue**:  
   - Maintains a **FIFO queue** of waiting threads (as `Node` objects) when they fail to acquire the state.
   - Threads in the queue are **parked** (blocked) using `LockSupport.park()` until signaled by another thread.

3. **Blocking and Waking**:  
   - When a thread cannot acquire the state (e.g., a lock is held), AQS **parks** it (via `LockSupport.park()`), which transitions it to the `WAITING` or `TIMED_WAITING` state.
   - When the state is released (e.g., a lock is unlocked), AQS **unparks** waiting threads (via `LockSupport.unpark()`) to retry acquiring the state.


### **AQS 对 Thread Scheduling 的影响**
AQS indirectly affects thread scheduling by:
1. **Reducing Spurious Wakeups and Busy-Waiting**:  
   - Instead of spinning (busy-waiting), AQS parks threads, allowing the OS scheduler to skip them until unparked. This reduces CPU usage and contention.

2. **Fairness Control**:  
   - AQS supports both **fair** and **unfair** modes:
     - **Fair mode**: Threads are granted access in FIFO order (queued). This ensures fairness but may reduce throughput due to frequent context switches.
     - **Unfair mode**: Threads may "bypass" the queue if the state is available, improving throughput but risking starvation for waiting threads.

3. **Thread Blocking/Waking Coordination**:  
   - AQS ensures that threads are woken up only when the synchronization state changes (e.g., a lock is released). This avoids unnecessary wakeups (unlike `Object.notify()` in intrinsic locks).

4. **Interruptible and Timed Waits**:  
   - AQS allows threads to respond to interrupts (`InterruptedException`) or timeout during waits (e.g., `tryLock(long timeout, TimeUnit)`), giving applications finer control over thread behavior.


### **Comparison to Intrinsic Locks (`synchronized`)**
| Feature                  | AQS (e.g., `ReentrantLock`)         | Intrinsic Lock (`synchronized`)       |
|--------------------------|--------------------------------------|----------------------------------------|
| Blocking Mechanism       | Uses `LockSupport.park()`            | Uses `Object.wait()`/`notify()`        |
| Fairness                 | Configurable (fair/unfair)           | No fairness guarantees                 |
| Interruptibility         | Supports `InterruptedException`      | Cannot interrupt waiting threads       |
| Timed Waits             | Supports timeouts (e.g., `tryLock`)  | No timeout support                     |
| Performance              | Optimized for high contention       | Slower under high contention           |

AQS-based synchronizers (like `ReentrantLock`) are generally more efficient and flexible than intrinsic locks, especially under high contention.


### **Underlying OS Interaction**
- **Parking/Unparking**:  
  `LockSupport.park()` and `unpark()` are implemented using OS-specific primitives:
  - On Linux: Uses `futex` (fast userspace mutex).
  - On Windows: Uses `WaitOnAddress` or `Condition Variables`.
  - These mechanisms allow threads to `sleep/wakeup` efficiently, `managed by the OS scheduler`.

- **Context Switching**:  
  When a thread is parked, the OS scheduler `removes` it from the `runnable queue`. 
  When unparked, it is `re-added` to the queue, triggering a context switch if necessary.


### **Example: `ReentrantLock` and AQS**
```java
ReentrantLock lock = new ReentrantLock();
lock.lock();  // Acquires the lock (may block)
try {
    // Critical section
} finally {
    lock.unlock();  // Releases the lock, unparks waiting threads
}
```
- If the lock is unavailable, `lock()` calls `AQS.acquire()`, which:
  1. Attempts to acquire the state (e.g., sets `state = 1` for the first lock).
  2. Fails → Adds the thread to the AQS queue.
  3. Parks the thread (OS scheduler skips it). 线程放弃 CPU，去休眠
- When `unlock()` is called:
  1. Releases the state (e.g., sets `state = 0`).
  2. Unparks the next waiting thread in the queue.


### **AQS 要点理解**
- **AQS does not replace the OS scheduler** but works *with* it to manage thread blocking/waking efficiently.
- Applications using AQS-based synchronizers can:
  - `Reduce contention` by parking threads instead of spinning. 减少竞争, 让线程去 sleep
  - Control fairness and timeouts. 控制获取锁的公平性和超时时间
  - Avoid deadlocks via interruptible waits. 可中断等待避免死锁
- The OS still schedules threads when they are `unparked`, but AQS minimizes unnecessary scheduling overhead by managing the `wait queue`. 被唤醒后依旧由 os 负责调度

In essence, **AQS abstracts the complexity of thread coordination**, while the OS scheduler handles the `actual execution of threads` based on their state (runnable, parked, etc.).

---

## synchronized 关键字和 OS 关系

**Intrinsic locks** (also known as **monitor locks**) are Java's built-in synchronization mechanism, implemented using the `synchronized` keyword. They provide mutual exclusion and visibility guarantees but are **less flexible and lower-level** compared to synchronizers like `AbstractQueuedSynchronizer` (AQS).


### **How Intrinsic Locks Work**
1. **Implicit Locking**:  
   `Every Java object` has an intrinsic lock (monitor). When a thread enters a `synchronized` method or block:
   - It **acquires the intrinsic lock** associated with the object (e.g., `this` for instance methods, the `class object` for static methods).
   - It **releases the lock** `automatically` when exiting the method/block (even if an exception is thrown).

2. **Blocking Behavior**:  
   - If a thread cannot acquire the lock (e.g., it's held by another thread), it **blocks** until the lock becomes available.
   - The JVM uses OS-specific mechanisms (e.g., `Object.wait()`/`notify()` internally) to manage blocked threads.

3. **Reentrancy**:  
   - Threads can reacquire the same lock multiple times (e.g., nested `synchronized` calls). The lock is released only when the thread exits the outermost `synchronized` block/method.


### **synchronized 对 Thread Scheduling 的影响**
Intrinsic locks interact with the OS scheduler similarly to AQS-based synchronizers, but with key differences:

| **Aspect**               | **Intrinsic Locks (`synchronized`)**          | **AQS-Based Locks (e.g., `ReentrantLock`)** |
|--------------------------|-----------------------------------------------|---------------------------------------------|
| **Blocking Mechanism**   | Uses `Object.wait()`/`notify()` internally    | Uses `LockSupport.park()`/`unpark()`        |
| **Fairness**             | No fairness guarantees                        | Configurable (fair/unfair)                  |
| **Interruptibility**     | Cannot interrupt waiting threads              | Supports `InterruptedException`             |
| **Timed Waits**          | No timeout support                            | Supports timeouts (e.g., `tryLock()`)       |
| **Performance**          | Optimized in modern JVMs (biased locking)     | More efficient under high contention        |


### **Limitations of Intrinsic Locks**
1. **No Fairness Control**:  
   - Threads waiting for the lock may be chosen arbitrarily by the JVM/OS, leading to potential **starvation** in high-contention scenarios.

2. **No Support for Try/Lock with Timeout**:  
   - Threads cannot attempt to acquire the lock without blocking (e.g., `tryLock()`), nor can they specify a timeout.

3. **No Interruption Handling**:  
   - Threads blocked on a `synchronized` lock **cannot be interrupted** (unlike AQS-based locks). This can lead to **deadlocks** if a thread holding the lock hangs or fails to release it.

4. **Coarse-Grained Control**:  
   - Limited to mutual exclusion; no support for advanced synchronization patterns (e.g., read/write locks, condition variables with signal groups).


### **Example: Intrinsic Lock Usage**
```java
public class Counter {
    private int count = 0;

    // Acquires the intrinsic lock on 'this'
    public synchronized void increment() {
        count++;
    }

    // Acquires the intrinsic lock on 'this'
    public synchronized int getCount() {
        return count;
    }
}
```
- If one thread is executing `increment()`, others must wait until it exits.
- No way to interrupt or time out the wait.



### **When to Use Intrinsic Locks**
- **Simple Use Cases**: For low-contention scenarios where simplicity and brevity are prioritized.
- **Legacy Code**: Older Java codebases often use `synchronized` due to historical reasons.
- **Performance**: Modern JVMs optimize intrinsic locks heavily (e.g., **biased locking**, **lock coarsening**), making them competitive with AQS-based locks in many cases.


### **OS Scheduler Interaction**
- When a thread blocks on an intrinsic lock, the JVM requests the OS to **park the thread** (similar to AQS, but via different mechanisms like `Object.wait()`).
- The OS scheduler `removes the thread from the runnable queue` until the lock is released and `notify()` is called.
- `Context switches` occur when threads are woken up, just like with AQS. 涉及到了内核


### **synchronized 要点理解**
- **Intrinsic locks are simpler but less flexible** than AQS-based synchronizers.
- **AQS provides finer control** over `fairness`, `timeouts`, and `interruption handling`, making it better suited for complex concurrency scenarios.
- **Modern JVMs optimize intrinsic locks**, so performance differences are often negligible unless dealing with high contention or requiring advanced features.

For most new applications, prefer **AQS-based utilities** (`ReentrantLock`, `Semaphore`, etc.) unless simplicity is critical. For deeper control, use `java.util.concurrent` abstractions built on AQS.

---

## `synchronized` 用户空间优化思路

The **core ideas** behind user-space optimizations for Java intrinsic locks (`synchronized`) revolve around **minimizing the cost of synchronization** by reducing reliance on expensive OS-level operations (like system calls or context switches). These optimizations exploit patterns in real-world code and runtime adaptability to make synchronization faster in common scenarios, while still falling back to kernel-level mechanisms only when necessary.


### **User-Space Lock 优化原则**
#### 1. **Avoid Kernel Transitions (Fast Paths)** 避免进入内核模式，尽量在用户空间获取锁
   - **Goal**: Reduce or eliminate transitions to the OS kernel (e.g., `futex` waits, mutex operations) for uncontended locks.
   - **How**:
     - Use **atomic instructions** (e.g., Compare-and-Swap, or CAS) in user space for lock acquisition.
     - Only escalate to OS-level blocking (e.g., `park()`, `wait()`) when contention is detected.
   - **Example**: Lightweight locking avoids syscalls for uncontended locks.

#### 2. **Leverage Runtime Profiling and Adaptivity**
   - **Goal**: Dynamically adapt lock behavior based on observed runtime patterns.
   - **How**:
     - The JVM's `Just-In-Time (JIT)` compiler and runtime monitor lock usage (e.g., frequency of contention, thread ownership).
     - Apply optimizations like **biased locking** or **adaptive spinning** based on runtime data.
   - **Example**: Biased locks are revoked only when contention occurs, saving overhead in single-threaded cases.

#### 3. **Reduce Lock Granularity Overhead** 减小锁的粒度和获取锁的次数
   - **Goal**: Minimize the number of lock operations and their critical sections.
   - **How**:
     - Merge adjacent or closely spaced `synchronized` blocks (**lock coarsening**).
     - Eliminate locks entirely for thread-local objects (**lock elision** via escape analysis).
   - **Trade-off**: Balance between reducing overhead and avoiding overly large critical sections that increase contention.

#### 4. **Exploit Common Concurrency Patterns** 获取锁的条件假设
   - **Goal**: Optimize for typical usage patterns in Java applications.
   - **How**:
     - Assume locks are often **uncontended** (e.g., lightweight locking).
     - Assume threads may **reacquire the same lock** (e.g., biased locking for reentrant access).
     - Assume waits are often **short-lived** (e.g., adaptive spinning before parking).


### **优化类型和细节**
#### 1. **Lightweight Locking (Fast Path)**
   - **Uncontended Locks**: Acquire a lock using a single CAS operation. If successful, no OS interaction is needed.
   - **Contended Locks**: If CAS fails (another thread holds the lock), escalate to OS-level blocking (e.g., `futex` on Linux).
   - **Why It Works**: Most locks in practice are uncontended, so the fast path avoids costly syscalls.

#### 2. **Biased Locking (Deprecated in Java 15+)**
   - **Core Idea**: Bias a lock toward a thread to eliminate atomic operations for `reentrant` access.
   - **Mechanism**: 
     - Mark the object header with the `thread ID` of the first acquirer.
     - Subsequent acquisitions by the `same` thread require only a thread ID check (no CAS).
   - **Trade-off**: Fast for single-threaded access but adds overhead for revocation under contention.

#### 3. **Lock Coarsening**
   - **Core Idea**: Merge adjacent `synchronized` blocks to reduce lock/unlock overhead.
   - **Example**:
     ```java
     synchronized(lock) { doA(); }
     synchronized(lock) { doB(); }
     ```
     → Merged into:
     ```java
     synchronized(lock) { doA(); doB(); }
     ```
   - **Why It Helps**: Reduces the number of lock operations and avoids unnecessary context switches.

#### 4. **Lock Elision (Escape Analysis)**
   - **Core Idea**: Remove locks entirely if the locked object is proven to be thread-local.
   - **Mechanism**: 
     - The JIT compiler analyzes object scope to determine if it escapes the current thread.
     - If not, synchronization is removed.
   - **Example**:
     ```java
     void method() {
         Object localLock = new Object();
         synchronized(localLock) { /* Lock elided */ }
     }
     ```

#### 5. **Adaptive Spinning**
   - **Core Idea**: Spin (busy-wait) briefly before parking a blocked thread, assuming the lock may become available soon.
   - **How**:
     - The spin count adapts dynamically based on historical contention (e.g., spin longer if the lock was recently contended).
   - **Why It Helps**: Avoids the cost of parking/unparking threads for short waits.


### **userspace 为什么优化会有用?**
1. **Most Locks Are Uncontended**:
   - Studies show `~90% of locks are uncontended` in real-world applications. `Lightweight locking` and `biased locking` optimize this common case. 锁大多数时候没有竞争

2. **Thread Reentrancy**:
   - Applications often `reacquire the same lock` (e.g., recursive method calls). Biased locking reduces overhead for reentrant access. 获取锁后, 会再次获取同一个锁

3. **Short Critical Sections**:
   - Critical sections are often small (e.g., incrementing a counter). Adaptive spinning avoids context switches for short waits. 临界区很短, 没有获取到的锁的线程可以先自旋一会, 很大可能可以成功获取锁

4. **Thread-Local Data**:
   - Many objects are never shared across threads. Escape analysis eliminates synchronization overhead for such cases. 大多数对象在线程间不共享


### **Trade-offs and Limitations**
| **Optimization**       | **Pros**                          | **Cons**                          |
|------------------------|-----------------------------------|-----------------------------------|
| Lightweight Locking    | Fast uncontended case 不竞争快速获取锁           | Falls back to OS for contention 竞争 os 就要干预了  |
| Biased Locking         | Zero-cost `reacquisition` 重新获取锁没有消耗        | Revocation overhead under contention |
| Lock Coarsening        | `Reduces` lock operations 减少获取锁的操作        | May increase critical section size 临界区变长|
| Lock Elision           | Eliminates sync overhead        | Limited to thread-local objects   |
| Adaptive Spinning      | Avoids context switches  自旋一会, 也就是 busy-wait      | Wastes CPU cycles for long waits  浪费一些 CPU 时间|


### **结论**
The **core idea** is to **handle synchronization efficiently in user space** whenever possible, using runtime adaptability and pattern recognition to minimize reliance on the kernel. 在用户空间高效处理同步, 尽量不要惊动 OS 内核, 一旦涉及内核代价较高 <br/>

#### 基于哪些事实在优化?
- Most locks are uncontended.
- Threads often reacquire locks.
- Critical sections are small.

By focusing on these patterns, modern JVMs make intrinsic locks (`synchronized`) competitive with AQS-based locks for many use cases, while still providing the safety and simplicity of built-in synchronization. For high-contention scenarios or advanced features (e.g., fairness, timeouts), AQS-based locks remain superior.