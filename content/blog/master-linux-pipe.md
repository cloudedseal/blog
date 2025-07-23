---
date: '2025-05-25T18:26:01+08:00'
draft: false
title: 'Master Linux Pipe'
---

## file descriptor

> A file descriptor, or FD, is a positive integer that refers to an input/output source.  指向 I/O 源,不关心具体的源是什么, 这就是抽象

所谓 I/O redirection 不过是对指定的 FD(指针) 复制而已



## `sh` 系统调用追踪

### shell pipeline 流程

**complete lifecycle of a shell pipeline** (`cat /etc/passwd | wc -l`) executed via `sh`.

- ✅ `pipe2()` (pipe creation, kernel managed buffer, pipe[0] read pipe[1] write)  
- ✅ `clone()` (Linux process creation, replaces `fork()`)  
- ✅ `dup2()` (redirects stdin/stdout)  
- ✅ `execve()` (runs `cat` and `wc -l`)
- ✅ `wait4()` (`sh` wait process status change)


```sh
strace -f -tt -s 1000 -o pipe.log -e trace=pipe2,clone,execve,dup2,close,wait4 sh -c 'cat /etc/passwd | wc -l'

[shell] 43141 11:44:13.141393 execve("/usr/bin/sh", ["sh", "-c", "cat /etc/passwd | wc -l"], 0x7ffc18a3a610 /* 66 vars */) = 0
[shell] 43141 11:44:13.151773 close(3)          = 0
[shell] 43141 11:44:13.164758 close(3)          = 0 # shell 关闭 pipe read 端
[shell] 43141 11:44:13.198842 pipe2([3, 4], 0)  = 0 # sh 准备 pipe[read,write]
[shell] 43141 11:44:13.199757 clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD, child_tidptr=0x7bf8e1d19a10) = 43142        # sh 新创建的进程将执行 cat
[shell] 43141 11:44:13.203158 close(4)          = 0 # shell 关闭 pipe write 端
[ cat ] 43142 11:44:13.204173 close(3)          = 0
[ cat ] 43142 11:44:13.206577 dup2(4, 1)        = 1 # 将 stdout 指向 pipe 的 write 端
[ cat ] 43142 11:44:13.210697 close(4)          = 0
[ cat ] 43142 11:44:13.214337 execve("/usr/bin/cat", ["cat", "/etc/passwd"], 0x587549212128 /* 66 vars */) = 0 # 执行 cat
[shell] 43141 11:44:13.221827 clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD, child_tidptr=0x7bf8e1d19a10) = 43143        # sh 新创建的进程将执行 wc
[shell] 43141 11:44:13.224486 close(3 <unfinished ...>
[ wc  ] 43143 11:44:13.225520 dup2(3, 0 <unfinished ...> # 将 stdin 指向 pipe 的 read 端
[shell] 43141 11:44:13.225697 <... close resumed>) = 0
[shell] 43141 11:44:13.226745 close(-1 <unfinished ...>
[ wc  ] 43143 11:44:13.227157 <... dup2 resumed>) = 0
[shell] 43141 11:44:13.229491 <... close resumed>) = -1 EBADF (Bad file descriptor)
[ wc  ] 43143 11:44:13.231492 close(3 <unfinished ...>
[shell] 43141 11:44:13.232277 wait4(-1,  <unfinished ...> 
[ wc  ] 43143 11:44:13.232353 <... close resumed>) = 0
[ wc  ] 43143 11:44:13.233515 execve("/usr/bin/wc", ["wc", "-l"], 0x587549212158 /* 66 vars */) = 0 # 执行 wc
[ cat ] 43142 11:44:13.237591 close(3)          = 0
[ wc  ] 43143 11:44:13.247690 close(3)          = 0
[ cat ] 43142 11:44:13.252126 close(3)          = 0
[ wc  ] 43143 11:44:13.261880 close(3)          = 0
[ cat ] 43142 11:44:13.273608 close(3)          = 0
[ wc  ] 43143 11:44:13.280205 close(3)          = 0
[ cat ] 43142 11:44:13.285117 close(3 <unfinished ...>
[ wc  ] 43143 11:44:13.285306 close(3)          = 0
[ cat ] 43142 11:44:13.286279 <... close resumed>) = 0
[ cat ] 43142 11:44:13.287263 close(1)          = 0
[ cat ] 43142 11:44:13.288559 close(2)          = 0
[ cat ] 43142 11:44:13.290401 +++ exited with 0 +++
[shell] 43141 11:44:13.290534 <... wait4 resumed>[{WIFEXITED(s) && WEXITSTATUS(s) == 0}], 0, NULL) = 43142 # sh 等待 cat
[shell] 43141 11:44:13.291286 --- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=43142, si_uid=1000, si_status=0, si_utime=0, si_stime=0} ---
[shell] 43141 11:44:13.293119 wait4(-1,  <unfinished ...>
[ wc  ] 43143 11:44:13.295141 close(3)          = 0
[ wc  ] 43143 11:44:13.301529 close(0)          = 0
[ wc  ] 43143 11:44:13.302466 close(1)          = 0
[ wc  ] 43143 11:44:13.303521 close(2)          = 0
[ wc  ] 43143 11:44:13.309383 +++ exited with 0 +++
[shell] 43141 11:44:13.311084 <... wait4 resumed>[{WIFEXITED(s) && WEXITSTATUS(s) == 0}], 0, NULL) = 43143 # sh 等待 wc
[shell] 43141 11:44:13.311821 --- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=43143, si_uid=1000, si_status=0, si_utime=0, si_stime=0} ---
[shell] 43141 11:44:13.314447 wait4(-1, 0x7fffd5a3124c, WNOHANG, NULL) = -1 ECHILD (No child processes)
[shell] 43141 11:44:13.318626 +++ exited with 0 +++

```

> By default, all open file descriptors remain open across an execve call unless explicitly marked with the `FD_CLOEXEC` flag.



### **数据流向**
1. **Pipe Setup (`shell`)**:
   - `pipe2([3,4])` creates a communication channel:
     - `3`: **Read** end (used by `wc`) read port(读口)
     - `4`: **Write** end (used by `cat`) write port(写口)

2. **Child 1 (`cat`)**:
   - `dup2(4, 1)` → Redirects `cat`'s stdout to the pipe 写口.
   - `execve("cat", ...)` → `cat` writes `/etc/passwd` to the pipe.

3. **Child 2 (`wc`)**:
   - `dup2(3, 0)` → Redirects `wc`'s stdin to the pipe 读口.
   - `execve("wc", ...)` → `wc` reads from the pipe and counts lines.

4. **Parent Cleanup**:
   - Closes both ends of the pipe (`close(3)` and `close(4)`) to prevent resource leaks.

### **数据流可视化**

![pipe2](https://raw.githubusercontent.com/cloudedseal/pictures/main/img/pipe2.drawio.svg)

```
+-------------------+
| Parent Shell      |
| (PID 43141)       |
+-------------------+
         |
         |  🔵 pipe2([3,4], 0)
         ▼
+-------------------+
| Pipe:             |
|   Read: FD 3      |
|   Write: FD 4     |
+-------------------+
         |
         |  🟢 clone(SIGCHLD) → PID 43142
         |  🟢 clone(SIGCHLD) → PID 43143
         ▼
+-------------------+       +-------------------+
| Child 43142 (cat) |       | Child 43143 (wc)  |
+-------------------+       +-------------------+
| 🟡 dup2(4, 1)     |       | 🟡 dup2(3, 0)      |
| 🔴 close(4)       |       | 🔴 close(3)        |
| 🟠 execve("cat")  |       | 🟠 execve("wc")    |
| → Writes to pipe  |       | → Reads from pipe |
+-------------------+       +-------------------+
         |                          ↑
         |                          |
         +--------------------------+
            Data flow via pipe
```

## pipeline 流程详细分析

### ✅ Step 1: Pipe Creation

```c
43141 11:44:13.198842 pipe2([3, 4], 0)  = 0
```

- The parent `shell` creates a pipe with two file descriptors:
  - `3`: Read end (for `wc`)
  - `4`: Write end (for `cat`)
- `pipe2()` is a modern variant of `pipe()` that supports flags. Here, no flags are set (`0`), so it behaves like `pipe()`.


### ✅ Step 2: Forking Child Processes

#### First Child (PID `43142`) – runs `cat`
```c
43141 11:44:13.199757 clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD, child_tidptr=0x7bf8e1d19a10) = 43142 
```

- The parent shell uses `clone()` to create a child process.
- This is how `fork()` is implemented in modern `glibc` — via `clone()` with `SIGCHLD`.

#### Second Child (PID `43143`) – runs `wc -l`
```c
43141 11:44:13.221827 clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD, child_tidptr=0x7bf8e1d19a10) = 43143
```

- The parent shell creates a second child process.
- Again, this is a `fork()` under the hood.



### ✅ Step 3: Redirecting File Descriptors

#### First Child (PID `43142`) – `cat`
```c
43142 11:44:13.206577 dup2(4, 1)        = 1 # 将 stdout 指向 pipe 的 write 端
43142 11:44:13.210697 close(4)          = 0
```

- Redirects stdout (`FD 1`) to the write end of the pipe (`FD 4`).
- Closes the redundant `FD 4` after duplication.

#### Second Child (PID `43143`) – `wc`
```c
43143 11:44:13.225520 dup2(3, 0 <unfinished ...>
43143 11:44:13.227157 <... dup2 resumed>) = 0
43143 11:44:13.231492 close(3 <unfinished ...>
```

- Redirects stdin (`FD 0`) to the read end of the pipe (`FD 3`).
- Closes the redundant `FD 3` after duplication.



### ✅ Step 4: Executing Commands

#### `cat` Process (PID `43142`)
```c
27641 13:23:49.476204 execve("/usr/bin/cat", ["cat", "/etc/passwd"], ...) = 0
```

- Replaces child process with `cat /etc/passwd`.
- Now `cat` writes its output to the pipe.

#### `wc` Process (PID `43143`)
```c
27642 13:23:49.483297 execve("/usr/bin/wc", ["wc", "-l"], ...) = 0
```

- Replaces child process with `wc -l`.
- Now `wc` reads input from the pipe and counts lines.



### ✅ Step 5: Parent Shell Cleanup

```c
43141 11:44:13.164758 close(3)          = 0
43141 11:44:13.203158 close(4)          = 0
```

- The parent shell closes both ends of the pipe.
- This ensures the reader (`wc`) knows when the writer (`cat`) has finished (when all writers have closed the pipe).



### ✅ Step 6: Process Exit

Both child processes exit cleanly:

```c
43142 11:44:13.290401 +++ exited with 0 +++

43143 11:44:13.309383 +++ exited with 0 +++
```

And finally, the parent shell exits:

```c
43141 11:44:13.318626 +++ exited with 0 +++
```

### ✅ Step 7: shell wait subProcess Exit


```c
43141 11:44:13.290534 <... wait4 resumed>[{WIFEXITED(s) && WEXITSTATUS(s) == 0}], 0, NULL) = 43142 #cat

43141 11:44:13.311084 <... wait4 resumed>[{WIFEXITED(s) && WEXITSTATUS(s) == 0}], 0, NULL) = 43143 #wc
```

And finally, the parent shell exits:

```c
43141 11:44:13.318626 +++ exited with 0 +++
```



## 涉及系统调用总结

| PID     | Action                                | Description |
|---------|----------------------------------------|-------------|
| 43141   | `pipe2([3, 4], 0)`                    | Creates pipe by shell process |
| 43141   | `clone(...)`                           | Forks first child (PID 27641) |
| 43141   | `clone(...)`                           | Forks second child (PID 27642) |
| 43142   | `dup2(4, 1)`                           | Redirects stdout to pipe write end |
| 43143   | `dup2(3, 0)`                           | Redirects stdin to pipe read end |
| 43142   | `execve("cat", ...)`                   | Replaces child with `cat` |
| 43143   | `execve("wc", ...)`                    | Replaces child with `wc -l` |
| 43141   | `wait4()`                               | Waits for both children to finish |




## References
1. [IO redirection in shell](https://thoughtbot.com/blog/input-output-redirection-in-the-shell)
2. [sys-call-clone](https://man7.org/linux/man-pages/man2/clone.2.html)
3. [sys-call-dup2](https://man7.org/linux/man-pages/man2/dup.2.html)
4. [sys-call-pipe2](https://man7.org/linux/man-pages/man2/pipe2.2.html)
5. [sys-call-execve](https://man7.org/linux/man-pages/man2/execve.2.html)
6. [sys-call-wait4](https://man7.org/linux/man-pages/man2/wait3.2.html)
7. [supported by qwen](https://chat.qwen.ai/c/83ce0a69-a113-4a13-89c4-9a50ce806806)
