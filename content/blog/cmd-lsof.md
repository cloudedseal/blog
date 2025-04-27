---
title: "lsof examples"
date: 2025-01-10T17:31:17+08:00
draft: false
---

## 通用
1. ^ 表示非
2. 多个值是 `CSV` 形式

## lsof -i(internet)
> -i 后可以有空格
> [46][protocol][@hostname|hostaddr][:service|port]
 
1. `46` specifies the IP version, IPv4 or IPv6 that applies to the following address.'6' may be be specified only if the UNIX dialect supports IPv6.  If neither '4' nor '6' is specified, the following address applies to all IP versions.  
2. `protocol` is a protocol name - TCP, UDP  
3. `hostname` is an Internet host name.  Unless a specific IP version is specified, open network files associated with host names of all versions will be selected.  
4. `hostaddr` is a numeric Internet IPv4 address in dot form; or an IPv6 numeric address in colon form enclosed in brackets, if the UNIX dialect supports IPv6.  When an IP version is selected, only its numeric addresses may be specified.  
5. `service` is an /etc/services name - e.g., smtp -or a list of them.  
6. `port` is a port number, or a list of them.


### lsof `-i`@10.10.129.99:8080 

### lsof `-i`4tcp

> all ipv4 tcp

### lsof `-i`4udp

> all ipv4 udp

### lsof `-i`4udp@localhost

> all localhost ipv4 udp

### lsof `-i`4tcp@10.10.129.99

> all 10.10.129.99 tcp 

### lsof `-i` :ssh

> service ssh

### lsof `-i`:22

> port 22

### lsof `-i`TCP -sTCP:LISTEN

> tcp state Listen


## lsof `-p` pid

> by pid

## lsof /test.sock 

## lsof `-t` /test.sock  

> -t(terse) 简要的 get pid

## lsof `-c` `^`java 

> 找到以非 java 命令(command)开始的

## lsof `-c` java 

> 找到以 java 命令(command)开始的

## lsof `+d` /dir

>  该 /dir目录(directory) 下所有

## lsof `-d` 1

> 查看 descriptor = 1(标准输出)

## lsof `-d` 1,2

> 查看 descriptor = 1,2(标准输出、标准错误)

## lsof `-a` `-c` java `-d` 3-10    
> 条件组合查询 -a(and) -c(command) -d(范围)

## lsof `-u` root

> 该 root 下所有 -u(user)

## lsof `-U`

> -U(Unix domain socket)
