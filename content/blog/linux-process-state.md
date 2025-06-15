---
title: "Linux Process State"
date: 2025-04-22T17:07:48+08:00
draft: false
tags: ["process","state","state machine"]
---

## 进程状态机

最近在看南京大学[蒋炎岩](https://ics.nju.edu.cn/~jyy/)老师的 2025 操作系统课程。里面有一句话, 计算机中的一切程序可以视为 `state machine`。很有启发。进程有初始状态, CPU 这个无情的执行指令的机器执行一条指令后，程序的状态就发生了变化。

![linux-process-state-machine-userspace](https://raw.githubusercontent.com/buybyte/pictures/main/img/linux-process-state-machine-userspace.drawio.svg)

## process state

### Running/Runnable(R)

万事俱备，只需要被 `scheduler` 调度到 CPU 上去运行。

### Interruptable Sleep(S)

处于这个状态的不会被 `scheduler` 调度到 CPU 上去运行。

#### d_state.c code

```c {filename="d_state.c"}
#include <unistd.h>

int main(){
    pause();
}
```
1. gcc -o d_state d_state.c
2. ./d_state
3. ps aux | grep d_state

### Uninterruptible Sleep(D)

可能是 `Disk Sleep` 的原因。状态用 `D` 表示。

### Stopped(T)

如使用 `SIGSTOP` 信号，暂停的进程。

#### d_state.c code

```c {filename="d_state.c"}
#include <unistd.h>

int main(){
    pause();
}
```
1. gcc -o d_state d_state.c
2. ./d_state
3. ps aux | grep d_state
4. kill -SIGTSTP $(pidof d_state)
5. ps aux | grep d_state


### Zombie(Z)

进程已经执行了 `exit`, `exit code` 还没有被`父进程` `wait/waitpid` 读取。也就是进程的 `PCB` 还没有从 kernel 的 `process table` 中清除。

#### zombie.c code

```c {filename="zombie.c"}
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sched.h>
int main() {
    pid_t child_pid = fork();

    if (child_pid == 0) {    // Child process
        printf("Child PID: %d\n", getpid());
        exit(0);             // Child exits immediately
    } else {                 // Parent process
        printf("Parent PID: %d\n", getpid());
        printf("Child PID: %d\n", child_pid);
        printf("Parent is sleeping (check for zombie with `ps`).\n");
        sleep(30);           // Parent sleeps (does not call wait())
        printf("Parent exiting.\n");
    }

    return 0;
}
```

1. gcc -o zombie zombie.c
2. ./zombie
3. ps aux | grep zombie

## system calls changing process state on Linux


### `fork()` 复制状态机器
• Purpose: Creates a new child process by duplicating the parent process.

• State Change: The child starts in the ready/runnable state (waiting for CPU time).

• Parameters: None.

• Return Value: 

  • `0` to the child process.

  • `Child's PID` to the parent.

  • `-1` on error.

• Example: Used in spawning new processes (e.g., shell commands).



### `execve()` 复位状态机
• Purpose: Replaces the current process's memory space with a new program.

• State Change: The process remains in the `running state` but executes new code.

• Parameters:

  • `path`: Path to the executable.

  • `argv`: Command-line arguments.

  • `envp`: Environment variables.

• Return Value: Only returns on error (`-1`).

• Example: Running programs like `ls` or `grep` from a shell.


### `exit()` / `_exit()` 销毁状态机
• Purpose: Terminates the current process.

• State Change: Moves the process to `zombie` (until the parent calls `wait()`).

• Parameters:

  • `status`: Exit status code.

• Return Value: None (does not return).

• Difference: `exit()` flushes buffers; `_exit()` is a raw system call.

• Example: Clean termination after program completion.


### `wait()` / `waitpid()` 等待子进程退出
• Purpose: Suspends the parent until a child changes state (exits or stops).

• State Change: Parent enters sleeping state until child exits.

• Parameters:

  • `status`: Stores child's exit status.

  • `pid`: Specific child to wait for (in `waitpid`).

• Return Value: Child PID on success; `-1` on error.

• Example: Parent process ensuring a child completes first.



### `kill()` / `raise()` 发送信号
• Purpose: Sends signals to processes (e.g., `SIGTERM`, `SIGSTOP`).

• State Change: 

  • `SIGSTOP`: Moves process to stopped state.

  • `SIGCONT`: Resumes a stopped process.

  • `SIGKILL`: Terminates immediately.

• Parameters:

  • `pid`: Target process ID.

  • `sig`: Signal to send.

• Return Value: `0` on success; `-1` on error.

• Example: Terminating a frozen process via `kill -9 PID`.


### `pause()` 暂停进程，直到收到信号
• Purpose: Suspends the process until a signal is received.

• State Change: Process enters **`interruptable sleeping`** state.

• Parameters: None.

• Return Value: Always `-1` (interrupted by signal).

• Example: Waiting indefinitely for user input or signals.


### `nanosleep()` 暂停进程一段时间
• Purpose: Pauses execution for a specified time.

• State Change: Process enters **`interruptible sleep`**.

• Parameters:

  • `req`: Time to sleep.

  • `rem`: Remaining time if interrupted.

• Return Value: `0` on success; `-1` on error.

• Example: High-precision delays in real-time applications.


### `ptrace()` 允许进程控制另一个进程
• Purpose: Allows a process to control another (debugging, tracing).

• State Change: Traced process enters **`stopped`** state on signals.

• Parameters:

  • `request`: Action (e.g., `PTRACE_ATTACH`, `PTRACE_CONT`).

  • `pid`: Target process ID.

• Return Value: Varies by request; `-1` on error.

• Example: Debuggers like `gdb` using breakpoints.


### `clone()`
• Purpose: Creates a child process or thread with configurable behavior.

• State Change: New process/thread starts in ready state.

• Parameters:

  • `fn`: Function for the child to execute.

  • `flags`: Options (e.g., `CLONE_VM` for shared memory).

  • `arg`: Arguments for `fn`.

• Return Value: Child PID on success; `-1` on error.

• Example: Implementing threads (used by `pthread` library).


### `sched_yield()` 主动放弃 CPU
• Purpose: Voluntarily yields the CPU to other processes/threads.

• State Change: Process moves from running to `ready` state.

• Parameters: None.

• Return Value: `0` on success; `-1` on error.

• Example: Cooperative multitasking in real-time apps.


### `exit_group()` 销毁进程中所有线程
• Purpose: Terminates all threads in a process.

• State Change: All threads enter zombie state.

• Parameters:

  • `status`: Exit code.

• Return Value: Does not return.

• Example: Terminating multi-threaded applications.


## Key Signals Affecting Process State
• `SIGSTOP`: Forcefully stops a process.

• `SIGCONT`: Resumes a stopped process.

• `SIGTERM`: Requests graceful termination.

• `SIGKILL`: Forcefully kills a process.


## Summary Table
| System Call     | Purpose                          | Key State Change               |
|-----------------|----------------------------------|---------------------------------|
| `fork()`        | Create child process             | Child → Runnable               |
| `execve()`      | Replace process image            | Process runs new code           |
| `exit()`        | Terminate process                | Process → Zombie               |
| `wait()`        | Wait for child state change      | Parent → Sleeping               |
| `kill()`        | Send signal (e.g., stop/resume)  | Target → Stopped/Running        |
| `pause()`       | Sleep until signal               | Process → Sleeping              |
| `ptrace()`      | Debug/trace another process      | Target → Stopped                |
| `clone()`       | Create thread/process            | New entity → Runnable           |

These system calls and signals form the backbone of process lifecycle management in Linux, enabling creation, termination, synchronization, and debugging of processes.