---
date: '2025-06-20T15:45:48+08:00'
draft: false
title: 'Linux File Descriptor'
---

## fd 一个编号,一个指针

指向文件 -> private_data 指向具体的文件, OS 内部对象

1. 最适合字节流，何为字节流，写进去就写进去了，读出来后就再也无法重新读出了。





## References

1. [file void *private_data 指向具体的文件，这就是一切皆为文件的实现](https://elixir.bootlin.com/linux/v6.15/source/include/linux/fs.h#L1064)
2. [file private_data 指向具体的文件 eventpoll](https://elixir.bootlin.com/linux/v6.15.4/source/fs/eventpoll.c#L179-L242)
3. [file private_data 指向具体的文件 socket](https://elixir.bootlin.com/linux/v6.15.4/source/include/linux/net.h#L107-L129)
4. [sock.h sk_data_ready 回调函数](https://elixir.bootlin.com/linux/v6.15.3/source/include/net/sock.h#L432)
5. [sock_def_readable](https://elixir.bootlin.com/linux/v6.15.3/source/net/core/sock.c#L3524)
6. [net.h socket_wq socket 等待队列](https://elixir.bootlin.com/linux/v6.15.3/source/include/linux/net.h#L99)
7. [sk_wait_event socket 进入阻塞状态](https://elixir.bootlin.com/linux/v6.15/source/include/net/sock.h#L1167)
8. [wait_woken 真正进入等待状态](https://elixir.bootlin.com/linux/v6.15/source/kernel/sched/wait.c#L413)
9. [socket.file socket 文件](https://elixir.bootlin.com/linux/v6.15.3/source/include/linux/net.h#L117)
10. [ip_input.c ip_rcv_finish_core 处理不同的协议](https://elixir.bootlin.com/linux/v6.15/source/net/ipv4/ip_input.c#L338)
11. [(source ip, source port, dest ip, dest port) 计算 hash 值](https://elixir.bootlin.com/linux/v6.15/source/net/ipv4/inet_hashtables.c#L32)
12. [file_operations 实现之 socket_file_ops](https://elixir.bootlin.com/linux/v6.15/source/net/socket.c#L156-L173)
13. [file_operations 实现之 ext4_file_operations](https://elixir.bootlin.com/linux/v6.15/source/fs/ext4/file.c#L962-L981)
14. [file_operations 实现之 pipe](https://elixir.bootlin.com/linux/v6.15/source/fs/pipe.c#L1243-L1263)