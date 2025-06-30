---
date: '2025-06-28T20:33:09+08:00'
draft: false
title: 'Linux User vs Kernel'
---

## 从 stack 说起

> 函数调用链使用 stack 实现的。所谓内核也是由一个一个内核的函数实现的。所以，毫无疑问，内核的函数调用链也是用 stack 实现的。

### 同一个线程，两个执行栈
1. user stack -> 执行用户的代码，比如 main, 在 `mm_struct` 结构中 [start_stack](https://elixir.bootlin.com/linux/v6.15/source/include/linux/mm_types.h#L1057), 栈大小用 ulimit -s 查看
2. kernel stack -> 执行内核代码, 比如 syscall, 在 `task_struct` 结构中的 [void *stack](https://elixir.bootlin.com/linux/v6.15/source/include/linux/sched.h#L832)
3. user stack ===>context switch ===>kernel stack

### 两个栈怎样切换?
**The same thread** executes both user-space code and kernel code. It transitions between modes via system calls but remains a single logical thread managed by the kernel.

When a thread transitions between `user` and `kernel` mode:
1. **execute code in user space**:
   - When a thread executes code in user space (e.g., `printf` formatting), it runs in **user mode** with limited privileges. 这就是所谓的线程的用户态
2. **CPU Automatically Switchs Stacks**:
   - On system calls (e.g., `write()`), exceptions (e.g., page faults), or hardware interrupts, the CPU:
     - Saves the `user-mode context` (thread's state: registers, instruction pointer, etc.) onto the **kernel stack**.
     - Switches to the kernel stack (predefined by the OS during thread creation). 
3. **Kernel Executes**:
   - The kernel runs the requested operation (e.g., writing to disk, handling an interrupt). 这就是所谓的线程的内核态
4. **Return to User Mode(非阻塞)**:
   - The kernel restores the user-mode context from the kernel stack and resumes execution in user space.
5. **Blocking Behavior(阻塞)**: 
   - If a system call blocks (e.g., waiting for disk I/O), the kernel marks the thread as `TASK_INTERRUPTIBLE` or `TASK_UNINTERRUPTIBLE`, and the scheduler `runs other` threads. When the I/O completes, the same thread resumes execution in user mode.


### **错误理解**
- **"Kernel Threads" vs. User Threads**:
  - Some kernel-internal threads (`kthreadd`,`ksoftirqd`) exist purely in kernel space and handle `background` tasks. These are unrelated to user-space threads.
  - Your program's threads are **user-space threads** that can enter kernel mode to request services but are not distinct from kernel threads. 用户的线程比如 main 线程可以进入 kernel mode，所谓内核态线程，但是线程实体是同一个。模式不同罢了。

### **write syscall 流程**
1. 在用户空间执行代码，那就是用户态线程
2. 在内核空间执行代码，那就是内核态线程
   
```plaintext
User Space                Kernel Space
-----------              ---------------
[Thread A]               [Scheduler]
   ↓                         ↓
User mode:                Kernel mode:
printf() → write() → sys_write()
   ↓                         ↓
[User stack]           [Kernel stack]
   ↓                         ↓
Return to user code ←  Return from syscall
```





## References

1. [kernel stack](https://elixir.bootlin.com/linux/v6.15/source/include/linux/sched.h#L832)
2. [user stack](https://elixir.bootlin.com/linux/v6.15/source/include/linux/mm_types.h#L1057)