---
date: '2025-05-23T11:55:31+08:00'
draft: false
title: 'Master Dns Resolve Steps'
---

> 分析环境 mint21

## linux 上 DNS 解析流程

- 以 ping -c 1 baidu.com 为例
- [ping strace 日志](https://github.com/cloudedseal/underlayer/blob/main/dns-resolve-expore/ping.log.9762)，分析 ping 如何拿到 baidu.com 的 IP, 向目标 IP 发送 ICMP

```bash
sudo strace -f -o ping.log  -e trace=execve,access,openat,socket,connect,newfstatat,sendmmsg,recvfrom,sendto,recvmsg,write  ping -c 1 baidu.com
```

### 1. **Start the `ping` Process**
   ```bash
   execve("/usr/bin/ping", ["ping", "-c", "1", "baidu.com"], 0x7ffce9f3de88 /* 26 vars */) = 0
   ```
   - The `ping` command begins execution.

### 2. **Load Required Libraries**
   ```bash
   openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
   openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libidn2.so.0", O_RDONLY|O_CLOEXEC) = 3
   ```
   - Libraries like `libc` (C standard library) and `libidn2` (Internationalized Domain Names) are loaded for DNS resolution and string handling. 处理国际化, 非英文字母的域名

### 3. **Check for `nscd` (Name Service Cache Daemon)** 检查缓存
   ```bash
   connect(5, {sa_family=AF_UNIX, sun_path="/var/run/nscd/socket"}, 110) = -1 ENOENT (No such file or directory)
   ```
   - The system tries to use `nscd` for cached DNS lookups but fails because `/var/run/nscd/socket` does not exist. This means caching is disabled or `nscd` is not running. 
   - 查看 `nscd` DNS 缓存, 这个服务默认没有安装

### 4. **Read `/etc/nsswitch.conf`** 获取 DNS 解析服务顺序
   ```bash
   openat(AT_FDCWD, "/etc/nsswitch.conf", O_RDONLY|O_CLOEXEC) = 5
   ```
   - The system reads the **Name Service Switch (NSS)** configuration. 
   - This file defines the `order of name resolution services` (e.g., `files` for `/etc/hosts`, `dns` for DNS). 
   - 定义了 DNS 解析服务的顺序

#### /etc/nsswitch.conf 内容

```bash
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

passwd:         files systemd
group:          files systemd
shadow:         files systemd
gshadow:        files systemd

hosts:          files mdns4_minimal [NOTFOUND=return] dns myhostname
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
```

##### 1. **`hosts:` Line**
```bash
hosts: files mdns4_minimal [NOTFOUND=return] dns myhostname
```
- **Order of resolution-DNS 解析核心逻辑**:
  1. **`files`**: Check `/etc/hosts` first (static hostname-to-IP mappings).
  2. **`mdns4_minimal`**: Use multicast DNS (mDNS) for `.local` names **IPv4-only** (common for local network devices).
     - `[NOTFOUND=return]`: If mDNS returns "not found," stop here and don’t proceed to `dns` or `myhostname`.
  3. **`dns`**: Use DNS (configured in `/etc/resolv.conf`).
  4. **`myhostname`**: Resolve the local machine’s hostname to its IP addresses (loopback and network interfaces).

- **Implications**:
  - **For regular domains (e.g., `baidu.com`)**: mDNS is skipped (since it’s not `.local`), so DNS is used directly.
  - **For `.local` names**: mDNS is prioritized, and DNS is ignored if mDNS fails.
  - **Security**: Prevents DNS fallback for mDNS queries, avoiding potential conflicts.

---

##### 2. **`passwd`, `group`, `shadow` Lines**
```bash
passwd: files systemd
group: files systemd
shadow: files systemd
```
- **Sources**:
  - **`files`**: Read from `/etc/passwd`, `/etc/group`, and `/etc/shadow`.
  - **`systemd`**: Integrates with `systemd` for dynamic users (e.g., containerized services or transient users).
- **Use Case**: Ensures compatibility with modern systems using `systemd`-managed users (e.g., Docker containers).

---

##### 3. **`netgroup` Line**
```bash
netgroup: nis
```
- **NIS (Network Information Service)**: Legacy protocol for centralized user/group management.
- **Warning**: NIS is insecure (plaintext communication). If unused, consider removing this line to avoid unnecessary risks.

---

#### **How This Affects DNS Resolution?**

##### 1. **DNS Lookup Path**
- For `baidu.com` (non-`.local` domain):
  1. Check `/etc/hosts` → No match.
  2. Skip `mdns4_minimal` (not relevant for `baidu.com`).
  3. Query DNS via `systemd-resolved` (uses `/etc/resolv.conf`).
  4. Fallback to `myhostname` only if DNS fails (unlikely).

##### 2. **`/etc/resolv.conf` Integration**
- The `dns` source uses settings from `/etc/resolv.conf` (e.g., `nameserver 127.0.0.53` for `systemd-resolved`).
- `systemd-resolved` acts as a local DNS stub resolver, handling:
  - DNSSEC validation.
  - Caching.
  - Split DNS (for VPNs).
  
---


### **Check `/etc/hosts`**
   ```bash
   openat(AT_FDCWD, "/etc/hosts", O_RDONLY|O_CLOEXEC) = 5
   ```
   - The system checks `/etc/hosts` for a static entry for `baidu.com`. 
   - If found, it skips DNS. No match is found here.
   - 在这个文件里寻找目标域名的 IP, 找到之后返回

### 5. **Read `/etc/resolv.conf`**

```bash
openat(AT_FDCWD, "/etc/resolv.conf", O_RDONLY|O_CLOEXEC) = 5
```

> 读取 DNS 解析器配置文件, 真正的配置在 /run/systemd/resolve/stub-resolv.conf

- dynamic DNS configuration file managed by `systemd-resolved`, the local DNS stub resolver. 
- It serves as the system's **primary DNS resolver** configuration, ensuring applications route DNS queries through systemd-resolved for features like `caching`, `DNSSEC validation`, and `split DNS` (e.g., for VPNs).

#### **`systemd-resolved`** local DNS stub resolver

```bash
systemctl status systemd-resolved.service
● systemd-resolved.service - Network Name Resolution
     Loaded: loaded (/usr/lib/systemd/system/systemd-resolved.service; enabled; preset: enabled)
     Active: active (running) since Sat 2025-05-24 15:09:18 CST; 58min ago
       Docs: man:systemd-resolved.service(8)
             man:org.freedesktop.resolve1(5)
             https://www.freedesktop.org/wiki/Software/systemd/writing-network-configuration-managers
             https://www.freedesktop.org/wiki/Software/systemd/writing-resolver-clients
   Main PID: 622 (systemd-resolve)
     Status: "Processing requests..."
      Tasks: 1 (limit: 4545)
     Memory: 7.5M (peak: 8.0M)
        CPU: 269ms
     CGroup: /system.slice/systemd-resolved.service
             └─622 /usr/lib/systemd/systemd-resolved
```

   
   

#### /etc/resolv.conf 内容
 
```bash
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0 trust-ad
search localdomain
```

##### 1. **`nameserver 127.0.0.53`**
- **Purpose**: All DNS queries from applications are sent to this **local DNS stub resolver**.
  - `127.0.0.53` is a special `loopback address` used exclusively by `systemd-resolved`.
  - The resolver listens on this address to handle queries from applications.
- **Why Not a Public DNS Server?**
  - Applications `don't communicate directly` with public DNS servers (e.g., `8.8.8.8`). Instead, they send queries to `127.0.0.53`, and `systemd-resolved` forwards them to upstream DNS servers configured elsewhere.
- **连接测试**
  ```bash
    nc -zv 127.0.0.53 53
    Connection to 127.0.0.53 53 port [tcp/domain] succeeded!
  ```

##### 2. **`options edns0 trust-ad`**
- **`edns0`**:
  - Enables **EDNS(0)** (Extension Mechanisms for DNS), allowing larger UDP packets (up to 4096 bytes instead of 512 bytes).
  - Required for DNSSEC and modern DNS features like DNS over TLS (DoT).
- **`trust-ad`**:
  - Tells the resolver to trust the **AD (Authentic Data)** bit in DNS responses.
  - When DNSSEC validation is enabled, `systemd-resolved` validates DNS responses and sets the AD bit if valid. This option ensures applications trust the resolver's validation results.

##### 3. **`search localdomain`**
- **Purpose**: Defines the **search domain** for unqualified hostnames (e.g., resolving `myhost` to `myhost.localdomain`).
- **Behavior**:
  - If you run `ping myhost`, the resolver will attempt:
    1. `myhost`
    2. `myhost.localdomain`
  - Avoids needing to type full FQDNs (Fully Qualified Domain Names) in local networks.


### 6. **Query DNS via `systemd-resolved`**
   ```bash
   socket(AF_INET, SOCK_DGRAM|SOCK_CLOEXEC|SOCK_NONBLOCK, IPPROTO_IP) = 5
   connect(5, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("127.0.0.53")}, 16) = 0
   sendmmsg(5, [{msg_hdr={...}, msg_len=38}, ...], 2, MSG_NOSIGNAL) = 2
   ```
   - A UDP socket is created to communicate with the DNS resolver (`127.0.0.53:53`).
   - Two DNS queries are sent:
     - **A record query** (IPv4 address) for `baidu.com`.
     - **AAAA record query** (IPv6 address) for `baidu.com`.


#### **upstream DNS servers** 上游的 DNS 服务器

> upstream DNS servers is external DNS servers that local DNS resolver `forwards` queries to for resolution

- The actual upstream DNS servers (e.g., your ISP's DNS or `1.1.1.1`) 
  1. `/etc/systemd/resolved.conf` (static configuration).
  2. Dynamically via `DHCP` or `VPN managers` (e.g., NetworkManager).

#### 查看 upstream DNS 解析器信息

```bash
resolvectl status
Global
         Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub

Link 2 (ens33)
    Current Scopes: DNS
         Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 172.16.222.2
       DNS Servers: 172.16.222.2
        DNS Domain: localdomain
```

> 注意 resolv.conf mode: stub


### 7. **Receive DNS Responses**
   ```bash
   recvfrom(5, "\266L\201\200\0\1\0\2\0\0\0\1\5baidu\3com\0\0\1\0\1\300\f\0\1\0...", 2048, 0, ...) = 70
   recvfrom(5, "\261e\201\200\0\1\0\0\0\1\0\1\5baidu\3com\0\0\34\0\1\300\f\0\6\0...", 65536, 0, ...) = 81
   ```
   - The DNS resolver returns:
     - **A record**: `39.156.66.10` (IPv4 address for `baidu.com`).
     - **AAAA record**: `2400:cb00:2048:1::681b:8093` (IPv6 address, but not shown in `strace` output).
   - The `ping` command uses the IPv4 address `39.156.66.10` for the ICMP echo request.

### 8. **Final Output**
   ```bash
   write(1, "PING baidu.com (39.156.66.10) 56"..., 52) = 52
   ```
   - The resolved IP address is printed, and `ping` sends an ICMP packet to `39.156.66.10`.

---

### **Key Files and Services Involved**
1. **`/etc/nsswitch.conf`**:
   - Controls the order of name resolution (e.g., `files` → `dns`).
2. **`/etc/resolv.conf`**:
   - Specifies DNS servers (e.g., `127.0.0.53` for `systemd-resolved`).
3. **`systemd-resolved`**:
   - A local DNS stub resolver that handles queries and caches results.
4. **`/etc/hosts`**:
   - Static hostname-to-IP mappings (bypasses DNS if a match exists).

---








## References

1. [systemd-resolved.service](https://www.man7.org/linux/man-pages/man8/systemd-resolved.service.8.html)
2. [nsswitch.conf](https://man7.org/linux/man-pages/man5/nsswitch.conf.5.html)
3. [gai.conf](https://man7.org/linux/man-pages/man5/gai.conf.5.html)
4. [resolv.conf](https://man7.org/linux/man-pages/man5/resolv.conf.5.html)
5. [resolvectl](https://www.man7.org/linux/man-pages/man1/resolvectl.1.html)
6. [Systemd-resolved](https://wiki.archlinux.org/title/Systemd-resolved)






