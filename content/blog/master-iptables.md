---
title: "Master Iptables"
date: 2025-01-07T11:15:57+08:00
draft: false
---

# 数据包
1. 接收
2. 发送
3. 转发 `路由器`

# iptables 三要素

## table
> tables allow you to do very `specific things` with packets. 对数据包做特定的处理
### filter table `default`

### mangle table

### nat table

### raw table



## chain
> each of these tables are `composed of a few default chains`. These chains allow you to `filter` packets at `various points`.

### PREROUTING

### INPUT

### OUTPUT

### FORWARD

### POSTROUTING

## target

### terminating target
1. 一旦匹配立刻决定一个网络包的最终命运。如 `ACCEPT`,`DROP`,`REJECT`
2. chain 上的 target 依次匹配, 一旦匹配就执行关联的操作。匹配不到执行默认的 target
3. 可以配置 default policy, 也是一个 target
4. 所有 chain 的 default policy 是 `ACCEPT`

### non-terminating target

1. 如 `LOG` 记录 kernel 日志

## iptables examples
> -t 不指定，默认是 filter table. 为啥? filter 常用呗

*N*ew-chain
*X*delete-chain
*L*ist
*I*nsert
*A*ppend
*D*elete
*R*eplace
*F*lush
*P*olicy
*j*ump
*m*atch *m*odule
*s*ource
*d*estination
*n*umeric 不进行 reverse dns lookup
*p*rotocol
*i*nput interface
*o*nput interface

### list rules

1. iptables -L --line-numbers

2. iptables -A INPUT -p tcp --dport 22 -j LOG
3. iptables -D INPUT -p tcp --dport 22 -j LOG

### append rules
1. iptables `-A` INPUT -s 221.194.47.0/24 -j REJECT
   
### delete rules
1. iptables `-D` INPUT -s 221.194.47.0/24 -j REJECT
2. iptables -D INPUT `12`
3. iptables -D INPUT `9`
4. iptables -F 
5. iptables -F INPUT

### insert/replace rules
1. iptables -I INPUT 1 -s 114.114.114.114 -j ACCEPT
2. iptables -R INPUT 1 -s 59.45.175.10 -J ACCEPT

### protocals/modules

1. iptables -A INPUT -p tcp -m tcp -dport 22 -s 59.45.175.0/24 -j DROP
2. iptables -A INPUT -p tcp -m multiport -dports 22,49101 -s 59.45.175.0/24 -j DROP
3. iptables -A input -p icmp -m icmp -icmp-type 17 -j DROP

### connection tracking module

### change default policy
1. iptables -P INPUT DROP
2. iptables -t nat -P INPUT DROP

### custom chain
1. iptables -N ssh-rules
2. iptables -A ssh-rules -s 18.130.0.0./16 -j ACCEPT
3. iptables -A ssh-rules -s 18.11.0.0/16 -j ACCEPT
4. iptables -A ssh-rules -j DROP
5. iptables -A INPUT -p tcp -m tcp --dport 22 -j `ssh-rules`


## 默认的 iptables 配置

> 所有 chain 的默认 policy 都是 ACCEPT

```bash
[root@node96 ~]# iptables -t filter -L -n --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination 

[root@node96 ~]# iptables -t nat -L -n --line-numbers
Chain PREROUTING (policy ACCEPT)
num  target     prot opt source               destination         

Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination         

Chain POSTROUTING (policy ACCEPT)
num  target     prot opt source               destination

[root@node96 ~]# iptables -t mangle -L -n --line-numbers
Chain PREROUTING (policy ACCEPT)
num  target     prot opt source               destination         

Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination         

Chain POSTROUTING (policy ACCEPT)
num  target     prot opt source               destination 

[root@node96 ~]# iptables -t raw -L -n --line-numbers
Chain PREROUTING (policy ACCEPT)
num  target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination 
```


# References
1. [depth-guide-iptables-linux-firewall-强烈推荐](https://www.booleanworld.com/depth-guide-iptables-linux-firewall/)
2. [iptables-解析](https://mp.weixin.qq.com/s/O084fYzUFk7jAzJ2DDeADg)
3. [iptables](https://man7.org/linux/man-pages/man8/iptables.8.html)
4. [iptables-extensions](https://man7.org/linux/man-pages/man8/iptables-extensions.8.html)
5. [ip-forward](https://www.linode.com/docs/guides/linux-router-and-ip-forwarding/#enable-ip-forwarding)
6. [https://manned.org/iptables](https://manned.org/iptables)