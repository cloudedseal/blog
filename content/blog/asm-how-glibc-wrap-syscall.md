---
title: "Asm How Glibc Wrap Syscall"
date: 2025-04-21T15:34:28+08:00
draft: false
tags: ["syscall","glibc","asm"]
---

> 实验平台: x86_64 GNU/Linux mint22.1

## 使用 `glibc` 的函数

write1.c

```c
#include <unistd.h>
int main()
{
    const char msg[] = "Hello, glibc!\n";
    write(1, msg, sizeof(msg) - 1); // glibc's write() wrapper
    return 0;
}
```
1. gcc -o write1 write1.c
2. 使用 `ltrace ./write1` 查看调用了 write 库函数

## 不使用 glibc 的函数

write2.c

```c
#include <errno.h>
#include <syscall.h>
#include <unistd.h>

ssize_t write_no_glibc(int fd, const void *buf, size_t count)
{
    long ret;
    asm volatile(
        "syscall"
        : "=a"(ret)                                      // Output: result in rax
        : "a"(__NR_write), "D"(fd), "S"(buf), "d"(count) // Inputs
        : "rcx", "r11", "memory"                         // Clobbered registers
    );

    if (ret < 0)
    {
        errno = -ret; // Set errno on error
        return -1;
    }
    return ret; // Return bytes written
}

int main()
{
    const char msg[] = "Hello, glibc!\n";
    write_no_glibc(1, msg, sizeof(msg) - 1);
    return 0;
}

```

1. gcc -o write2 write2.c
2. 使用 `ltrace ./write2` 查看没有调用 write 库函数

## How glibc Wraps System Calls​​

### ​System Call Number​​

Each system call (e.g., write, read) is assigned a `unique number` (e.g., __NR_write). 要使用哪一个 syscall

### Argument Setup​​ 
The wrapper loads the system call number and arguments into specific registers (architecture-dependent). 设置 syscall 的参数

### ​Kernel Transition​​ 
The wrapper uses an instruction like syscall (x86-64) to switch to kernel mode. 进入 kernel mode

### Result Handling​​
After the kernel finishes, the wrapper checks for errors, sets errno if needed, and returns the result. 处理返回值









## References
1. [https://man7.org/linux/man-pages/man1/ltrace.1.html](https://man7.org/linux/man-pages/man1/ltrace.1.html)

