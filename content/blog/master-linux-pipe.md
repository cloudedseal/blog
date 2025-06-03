---
date: '2025-05-25T18:26:01+08:00'
draft: false
title: 'Master Linux Pipe'
---

## file descriptor

> A file descriptor, or FD, is a positive integer that refers to an input/output source.  指向 I/O 源,不关心具体的源是什么, 这就是抽象

所谓 I/O redirection 不过是对指定的 FD 复制而已



## `sh` 系统调用追踪

### shell pipeline 流程

**complete lifecycle of a shell pipeline** (`cat /etc/passwd | wc -l`) executed via `sh`.

- ✅ `pipe2()` (pipe creation, kernel managed buffer, pipe[0] read pipe[1] write)  
- ✅ `clone()` (Linux process creation, replaces `fork()`)  
- ✅ `dup2()` (redirects stdin/stdout)  
- ✅ `execve()` (runs `cat` and `wc -l`)  


```sh
strace -f -tt -s 1000 -o pipe.log -e trace=pipe2,clone,execve,dup2,close sh -c 'cat /etc/passwd | wc -l'

27640 13:23:49.436915 execve("/usr/bin/sh", ["sh", "-c", "cat /etc/passwd | wc -l"], 0x7ffcc4e072a0 /* 65 vars */) = 0
27640 13:23:49.446359 close(3)          = 0
27640 13:23:49.452650 close(3)          = 0
27640 13:23:49.471471 pipe2([3, 4], 0)  = 0
27640 13:23:49.472226 clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD, child_tidptr=0x77cc40b30a10) = 27641
27640 13:23:49.474244 close(4)          = 0
27641 13:23:49.474851 close(3)          = 0
27641 13:23:49.475308 dup2(4, 1)        = 1
27641 13:23:49.475737 close(4)          = 0
27641 13:23:49.476204 execve("/usr/bin/cat", ["cat", "/etc/passwd"], 0x6292fba9d0f8 /* 65 vars */ <unfinished ...>
27640 13:23:49.477420 clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD <unfinished ...>
27641 13:23:49.477659 <... execve resumed>) = 0
27640 13:23:49.478608 <... clone resumed>, child_tidptr=0x77cc40b30a10) = 27642
27640 13:23:49.479006 close(3)          = 0
27640 13:23:49.480741 close(-1 <unfinished ...>
27642 13:23:49.480811 dup2(3, 0)        = 0
27640 13:23:49.481370 <... close resumed>) = -1 EBADF (Bad file descriptor)
27642 13:23:49.482849 close(3)          = 0
27642 13:23:49.483297 execve("/usr/bin/wc", ["wc", "-l"], 0x6292fba9d128 /* 65 vars */) = 0
27641 13:23:49.486693 close(3)          = 0
27642 13:23:49.487874 close(3)          = 0
27642 13:23:49.491607 close(3)          = 0
27641 13:23:49.494576 close(3)          = 0
27642 13:23:49.500190 close(3)          = 0
27642 13:23:49.502045 close(3)          = 0
27642 13:23:49.504291 close(3)          = 0
27641 13:23:49.507177 close(3)          = 0
27641 13:23:49.515626 close(3)          = 0
27641 13:23:49.516854 close(1)          = 0
27641 13:23:49.517974 close(2)          = 0
27642 13:23:49.519465 close(0 <unfinished ...>
27641 13:23:49.519562 +++ exited with 0 +++
27642 13:23:49.519778 <... close resumed>) = 0
27642 13:23:49.520356 close(1 <unfinished ...>
27640 13:23:49.520949 --- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=27641, si_uid=1000, si_status=0, si_utime=0, si_stime=0} ---
27642 13:23:49.521080 <... close resumed>) = 0
27642 13:23:49.521383 close(2)          = 0
27642 13:23:49.523033 +++ exited with 0 +++
27640 13:23:49.523476 --- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=27642, si_uid=1000, si_status=0, si_utime=0, si_stime=0} ---
27640 13:23:49.527449 +++ exited with 0 +++

```

### strace 过滤

```bash
grep -E 'pipe2|clone|dup2|execve' pipe.log

27640 13:23:49.436915 execve("/usr/bin/sh", ["sh", "-c", "cat /etc/passwd | wc -l"], 0x7ffcc4e072a0 /* 65 vars */) = 0
27640 13:23:49.471471 pipe2([3, 4], 0)  = 0
27640 13:23:49.472226 clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD, child_tidptr=0x77cc40b30a10) = 27641
27641 13:23:49.475308 dup2(4, 1)        = 1
27641 13:23:49.476204 execve("/usr/bin/cat", ["cat", "/etc/passwd"], 0x6292fba9d0f8 /* 65 vars */ <unfinished ...>
27640 13:23:49.477420 clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD <unfinished ...>
27641 13:23:49.477659 <... execve resumed>) = 0
27640 13:23:49.478608 <... clone resumed>, child_tidptr=0x77cc40b30a10) = 27642
27642 13:23:49.480811 dup2(3, 0)        = 0
27642 13:23:49.483297 execve("/usr/bin/wc", ["wc", "-l"], 0x6292fba9d128 /* 65 vars */) = 0

```

#### 进程 ID

- **Parent Shell (sh)**: PID `27640`
- **Child 1 (`cat`)**: PID `27641`
- **Child 2 (`wc`)**: PID `27642`


### **数据流向**
1. **Pipe Setup**:
   - `pipe2([3,4])` creates a communication channel:
     - `3`: **Read** end (used by `wc`)
     - `4`: **Write** end (used by `cat`)

2. **Child 1 (`cat`)**:
   - `dup2(4, 1)` → Redirects `cat`'s stdout to the pipe.
   - `execve("cat", ...)` → `cat` writes `/etc/passwd` to the pipe.

3. **Child 2 (`wc`)**:
   - `dup2(3, 0)` → Redirects `wc`'s stdin to the pipe.
   - `execve("wc", ...)` → `wc` reads from the pipe and counts lines.

4. **Parent Cleanup**:
   - Closes both ends of the pipe (`close(3)` and `close(4)`) to prevent resource leaks.

### **数据流可视化**

```
+-------------------+
| Parent Shell      |
| (PID 27640)       |
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
         |  🟢 clone(SIGCHLD) → PID 27641
         |  🟢 clone(SIGCHLD) → PID 27642
         ▼
+-------------------+       +-------------------+
| Child 27641 (cat) |       | Child 27642 (wc)  |
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
27640 13:23:49.471471 pipe2([3, 4], 0) = 0
```

- The parent shell creates a pipe with two file descriptors:
  - `3`: Read end (for `wc`)
  - `4`: Write end (for `cat`)
- `pipe2()` is a modern variant of `pipe()` that supports flags. Here, no flags are set (`0`), so it behaves like `pipe()`.


### ✅ Step 2: Forking Child Processes

#### First Child (PID `27641`) – runs `cat`
```c
27640 13:23:49.472226 clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD, ...) = 27641
```

- The parent shell uses `clone()` to create a child process.
- This is how `fork()` is implemented in modern `glibc` — via `clone()` with `SIGCHLD`.

#### Second Child (PID `27642`) – runs `wc -l`
```c
27640 13:23:49.478608 clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD, ...) = 27642
```

- The parent shell creates a second child process.
- Again, this is a `fork()` under the hood.



### ✅ Step 3: Redirecting File Descriptors

#### First Child (PID `27641`) – `cat`
```c
27641 13:23:49.475308 dup2(4, 1) = 1
27641 13:23:49.475737 close(4) = 0
```

- Redirects stdout (`FD 1`) to the write end of the pipe (`FD 4`).
- Closes the redundant `FD 4` after duplication.

#### Second Child (PID `27642`) – `wc`
```c
27642 13:23:49.480811 dup2(3, 0) = 0
27642 13:23:49.482849 close(3) = 0
```

- Redirects stdin (`FD 0`) to the read end of the pipe (`FD 3`).
- Closes the redundant `FD 3` after duplication.



### ✅ Step 4: Executing Commands

#### `cat` Process (PID `27641`)
```c
27641 13:23:49.476204 execve("/usr/bin/cat", ["cat", "/etc/passwd"], ...) = 0
```

- Replaces child process with `cat /etc/passwd`.
- Now `cat` writes its output to the pipe.

#### `wc` Process (PID `27642`)
```c
27642 13:23:49.483297 execve("/usr/bin/wc", ["wc", "-l"], ...) = 0
```

- Replaces child process with `wc -l`.
- Now `wc` reads input from the pipe and counts lines.



### ✅ Step 5: Parent Shell Cleanup

```c
27640 13:23:49.474244 close(4) = 0
27640 13:23:49.479006 close(3) = 0
```

- The parent shell closes both ends of the pipe.
- This ensures the reader (`wc`) knows when the writer (`cat`) has finished (when all writers have closed the pipe).



### ✅ Step 6: Process Exit

Both child processes exit cleanly:

```c
[pid 27641] +++ exited with 0 +++
[pid 27642] +++ exited with 0 +++
```

And finally, the parent shell exits:

```c
[pid 27640] +++ exited with 0 +++
```



## 涉及系统调用总结

| PID     | Action                                | Description |
|---------|----------------------------------------|-------------|
| 27640   | `pipe2([3, 4], 0)`                    | Creates pipe |
| 27640   | `clone(...)`                           | Forks first child (PID 27641) |
| 27640   | `clone(...)`                           | Forks second child (PID 27642) |
| 27641   | `dup2(4, 1)`                           | Redirects stdout to pipe write end |
| 27642   | `dup2(3, 0)`                           | Redirects stdin to pipe read end |
| 27641   | `execve("cat", ...)`                   | Replaces child with `cat` |
| 27642   | `execve("wc", ...)`                    | Replaces child with `wc -l` |
| 27640   | `wait()`                               | Waits for both children to finish |




## References
1. [IO redirection in shell](https://thoughtbot.com/blog/input-output-redirection-in-the-shell)
2. [sys-call-clone](https://man7.org/linux/man-pages/man2/clone.2.html)
3. [sys-call-dup2](https://man7.org/linux/man-pages/man2/dup.2.html)
4. [sys-call-pipe2](https://man7.org/linux/man-pages/man2/pipe2.2.html)
5. [sys-call-execve](https://man7.org/linux/man-pages/man2/execve.2.html)
6. [supported by qwen](https://chat.qwen.ai/c/83ce0a69-a113-4a13-89c4-9a50ce806806)
