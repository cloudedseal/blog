---
date: '2025-07-16T17:47:03+08:00'
draft: false
title: 'Linux Modify Virtual Memory'
---

## 使用的测试代码

```c{fileName="loop.c"}
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

/**
 * main - uses strdup to create a new string, loops forever-ever
 *
 * Return: EXIT_FAILURE if malloc failed. Other never returns
 */
int main(void)
{
     char *s;
     unsigned long int i;

     s = strdup("Hello virtual memory");
     if (s == NULL)
     {
          fprintf(stderr, "Can't allocate mem with malloc\n");
          return (EXIT_FAILURE);
     }
     i = 0;
     while (s)
     {
          printf("[%lu] %s (%p)\n", i, s, (void *)s);
          sleep(2);
          i++;
     }
     return (EXIT_SUCCESS);
}

```



## 修改 virtual Memory

1. gcc -o loop loop.c
   
### 使用 dd 修改 /proc/pid/mem

```bash

./loop 
[0] Hello virtual memory (0x5cdeb6ed72a0)
[1] Hello virtual memory (0x5cdeb6ed72a0)

# 更改 1 个字节的数据
echo -ne "\x42" | sudo dd of=/proc/$(pidof loop)/mem bs=1 count=1 seek=$((0x5cdeb6ed72a0)) conv=notrunc

# 更改 4 个字节的数据
echo -ne "\x41\x42\x43\x44" | sudo dd of=/proc/$(pidof loop)/mem bs=1 count=4 seek=$((0x5a6be9ed32a0)) conv=notrunc

# 更改 5 个字节的数据
echo -ne "\x41\x42\x43\x44\x00" | sudo dd of=/proc/$(pidof loop)/mem bs=1 count=5 seek=$((0x5a6be9ed32a0)) conv=notrunc


echo -ne "\x41\x42\x43\x44\x44" | sudo dd of=/proc/$(pidof loop)/mem bs=1 count=5 seek=$((0x5a6be9ed32a0)) conv=notrunc
```

### 使用 gdb 修改

```bash
sudo gdb -p $(pidof loop) -batch -ex "set {int}0x6515df2922a0 = 0x41424344" -ex "detach" -ex "quit"

sudo gdb -p $(pidof loop) -batch -ex "set {char}0x6515df2922a0 = 0x41" -ex "detach" -ex "quit"

```







## References

1. [hack-the-virtual-memory-c-strings-proc](https://blog.holbertonschool.com/hack-the-virtual-memory-c-strings-proc/)
2. [/proc/pid/mem](https://man7.org/linux/man-pages/man5/proc_pid_mem.5.html)
3. [supportedByQwen](https://chat.qwen.ai/c/71592756-3669-424d-8613-f40a26b7944a)
