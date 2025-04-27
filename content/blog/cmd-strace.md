---
title: "Cmd Strace"
date: 2025-04-19T16:29:04+08:00
draft: false
tags: ["syscall","strace"]
---

# syscall

操作系统运行在 kernel space, 拥有整个系统的控制权。
应用程序运行在  user  space, 拥有部分权限。
这就是隔离。
To prevent user applications from `accessing or modifying` critical operating system data.
想让操作系统做些事情怎么办？使用 syscall。

# strace


## strace -t -ff -o MainTest.log -f  java MainTest
1. `-t`  time
2. `-ff` follow-fork
3. `-o`  output

## strace -tt -ff -o MainTest.log  -f -e trace=futex,write java MainTest

> -ff -o 每一个进程的 log 单独写到一个文件

```bash
-rw-rw-r-- 1 yang yang  13468 Apr 20 21:10 MainTest.log.21792
-rw-rw-r-- 1 yang yang 455425 Apr 20 21:10 MainTest.log.21793
-rw-rw-r-- 1 yang yang   1254 Apr 20 21:10 MainTest.log.21794
-rw-rw-r-- 1 yang yang   1361 Apr 20 21:10 MainTest.log.21795
-rw-rw-r-- 1 yang yang   3802 Apr 20 21:10 MainTest.log.21796
-rw-rw-r-- 1 yang yang   1677 Apr 20 21:10 MainTest.log.21797
-rw-rw-r-- 1 yang yang   1705 Apr 20 21:10 MainTest.log.21798
-rw-rw-r-- 1 yang yang   1797 Apr 20 21:10 MainTest.log.21810
-rw-rw-r-- 1 yang yang   6087 Apr 20 21:10 MainTest.log.21811
-rw-rw-r-- 1 yang yang   5541 Apr 20 21:10 MainTest.log.21812
-rw-rw-r-- 1 yang yang   1444 Apr 20 21:10 MainTest.log.21813
-rw-rw-r-- 1 yang yang  33890 Apr 20 21:10 MainTest.log.21814
-rw-rw-r-- 1 yang yang   3996 Apr 20 21:10 MainTest.log.21822
```

## strace -tt -ff -o MainTest.log  -f -e trace=futex,write  java MainTest 

> -e expr 多个 syscall CSV 形式

## strace -tt -ff -o MainTest.log  -f -e trace=%process java MainTest

## strace -tt -ff -o MainTest.log  -f -e trace=%memory -a java MainTest 

## strace -tt -ff -o MainTest.log  -f -e trace=%ipc  java MainTest 


# References
1. [https://jvns.ca/strace-zine-v2.pdf](https://jvns.ca/strace-zine-v2.pdf)
2. [https://man7.org/linux/man-pages/man1/strace.1.html](https://man7.org/linux/man-pages/man1/strace.1.html)
3. [https://strace.io/](https://strace.io/)

