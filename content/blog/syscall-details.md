---
date: '2025-07-27T17:45:55+08:00'
draft: false
title: 'Syscall Details'
tags: ["syscall"]
---

### 什么是操作系统？

os 初始状态 + syscalls

When a user program issues a system call, the operating system (OS) and hardware collaborate to transition from user mode to kernel mode, execute `privileged` operations, and safely return control to the user program. 


### **1. Issuing the System Call**
+ **User Program Action**: The program invokes a system call (e.g., `read()` or `write()`) via a library function (e.g., `libc`).
+ **Setup**: The library function prepares:
    - **System Call Number**: Identifies the requested service (e.g., stored in `RAX` on x86-64).
    - **Arguments**: Passed via registers (e.g., `RDI`, `RSI`, `RDX` on x86-64).
+ **Trigger**: Executes a hardware instruction (e.g., `syscall` on x86-64, `svc` on ARM, or `int 0x80` for legacy x86) to switch to kernel mode.


### **2. Hardware Transition to Kernel Mode**
+ **Privilege Elevation**: The CPU switches from **user mode** (limited privileges) to **kernel mode** (full privileges).
+ **Interrupt Handling**:
    - The CPU consults the **Interrupt Descriptor Table (IDT)** or system call entry point to locate the kernel's handler.
    - The **Memory Management Unit (MMU)** enforces memory protection, ensuring user programs cannot access kernel memory.
+ **Context Save**:
    - The CPU saves the user program’s state (registers, program counter, stack pointer, flags) to the **kernel stack** (not the user stack).
    - The kernel stack is per-process and isolated for security.



### **3. Kernel Executes the System Call**
+ **Handler Invocation**: The kernel’s system call dispatcher uses the system call number to jump to the appropriate handler (e.g., `sys_read()` or `sys_write()`).
+ **Privileged Operations**:
    - The kernel performs tasks like I/O, file access, or memory allocation, interacting directly with hardware if needed.
    - If the operation blocks (e.g., waiting for disk data), the process may be paused, and the scheduler switches to another task.
+ **Result Preparation**: The return value (or error code) is stored in a register (e.g., `RAX` on x86-64).



### **4. Returning to User Mode**
+ **Context Restoration**: The kernel restores the saved user state (registers, program counter) from the kernel stack.
+ **Mode Switch**: The CPU executes a return instruction (e.g., `sysret` or `iret` on x86), switching back to **user mode**.
+ **Resumption**: The user program continues execution at the next instruction, typically checking the return value for success/error.



### **Hardware and OS Roles**
+ **CPU**: Manages privilege levels, mode transitions, and instruction execution.
+ **MMU**: Enforces memory isolation between user and kernel spaces.
+ **Kernel**: Provides system call handlers, manages resources, and ensures security.



### **Summary**
+ **User-to-Kernel Transition**: Hardware elevates privileges, saves state, and jumps to the kernel.
+ **Kernel Execution**: Performs the requested service using privileged operations.
+ **Kernel-to-User Transition**: Restores user state, drops privileges, and resumes the user program.

This controlled interaction ensures user programs can safely access OS services without compromising system stability or security.





# References
1. [https://kib.kiev.ua/x86docs/AMD/MISC/21086C.pdf](https://kib.kiev.ua/x86docs/AMD/MISC/21086C.pdf)

