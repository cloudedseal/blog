---
date: '2025-07-30T17:07:23+08:00'
draft: false
title: 'Master Dynamic Dns'
tags: ["dns", "nsupdate"]
---

## 重要概念

### dns zone
> A DNS zone is a portion of the DNS namespace that is managed by a specific organization or administrator.
> All of the information for a zone is stored in what’s called a `DNS zone file`, which is the key to understanding how a DNS zone operates.
> 一个 zone 就是由一个机构或者管理员管理的某一个 DNS namespace。所有的信息存在 DNS zone file 中。我的地盘我做主。

### dns zone file
>A zone file is a plain text file stored in a DNS server that contains an actual representation of the zone and contains all the records for every domain within the zone. Zone files must always start with a [Start of Authority (SOA) record](https://www.cloudflare.com/learning/dns/dns-records/dns-soa-record/), which contains important information including contact information for the zone administrator.

1. 文本文件
2. 包括该 zone 下每一个 domain 的所有 record
3. 必须以 SOA record 开始



## DNS服务-bind
`rpm -ql bind`

> /etc/logrotate.d/named  
/etc/named  
/etc/named.conf  
/etc/named.iscdlv.key  
/etc/named.rfc1912.zones  
/etc/named.root.key  
/etc/rndc.conf  
/etc/rndc.key  
/etc/rwtab.d/named  
/etc/sysconfig/named  
/run/named  
/usr/bin/arpaname  
/usr/bin/named-rrchecker  
/usr/lib/python2.7/site-packages/isc  
/usr/lib/python2.7/site-packages/isc-2.0-py2.7.egg-info  
/usr/lib/python2.7/site-packages/isc/__init__.py  
/usr/lib/python2.7/site-packages/isc/__init__.pyc  
/usr/lib/python2.7/site-packages/isc/__init__.pyo  
/usr/lib/python2.7/site-packages/isc/checkds.py  
/usr/lib/python2.7/site-packages/isc/checkds.pyc  
/usr/lib/python2.7/site-packages/isc/checkds.pyo  
/usr/lib/python2.7/site-packages/isc/coverage.py  
/usr/lib/python2.7/site-packages/isc/coverage.pyc  
/usr/lib/python2.7/site-packages/isc/coverage.pyo  
/usr/lib/python2.7/site-packages/isc/dnskey.py  
/usr/lib/python2.7/site-packages/isc/dnskey.pyc  
/usr/lib/python2.7/site-packages/isc/dnskey.pyo  
/usr/lib/python2.7/site-packages/isc/eventlist.py  
/usr/lib/python2.7/site-packages/isc/eventlist.pyc  
/usr/lib/python2.7/site-packages/isc/eventlist.pyo  
/usr/lib/python2.7/site-packages/isc/keydict.py  
/usr/lib/python2.7/site-packages/isc/keydict.pyc  
/usr/lib/python2.7/site-packages/isc/keydict.pyo  
/usr/lib/python2.7/site-packages/isc/keyevent.py  
/usr/lib/python2.7/site-packages/isc/keyevent.pyc  
/usr/lib/python2.7/site-packages/isc/keyevent.pyo  
/usr/lib/python2.7/site-packages/isc/keymgr.py  
/usr/lib/python2.7/site-packages/isc/keymgr.pyc  
/usr/lib/python2.7/site-packages/isc/keymgr.pyo  
/usr/lib/python2.7/site-packages/isc/keyseries.py  
/usr/lib/python2.7/site-packages/isc/keyseries.pyc  
/usr/lib/python2.7/site-packages/isc/keyseries.pyo  
/usr/lib/python2.7/site-packages/isc/keyzone.py  
/usr/lib/python2.7/site-packages/isc/keyzone.pyc  
/usr/lib/python2.7/site-packages/isc/keyzone.pyo  
/usr/lib/python2.7/site-packages/isc/parsetab.py  
/usr/lib/python2.7/site-packages/isc/parsetab.pyc  
/usr/lib/python2.7/site-packages/isc/parsetab.pyo  
/usr/lib/python2.7/site-packages/isc/policy.py  
/usr/lib/python2.7/site-packages/isc/policy.pyc  
/usr/lib/python2.7/site-packages/isc/policy.pyo  
/usr/lib/python2.7/site-packages/isc/rndc.py  
/usr/lib/python2.7/site-packages/isc/rndc.pyc  
/usr/lib/python2.7/site-packages/isc/rndc.pyo  
/usr/lib/python2.7/site-packages/isc/utils.py  
/usr/lib/python2.7/site-packages/isc/utils.pyc  
/usr/lib/python2.7/site-packages/isc/utils.pyo  
/usr/lib/systemd/system/named-setup-rndc.service  
/usr/lib/systemd/system/named.service  
/usr/lib/tmpfiles.d/named.conf  
/usr/lib64/bind  
/usr/libexec/generate-rndc-key.sh  
/usr/sbin/ddns-confgen  
/usr/sbin/dnssec-checkds  
/usr/sbin/dnssec-coverage  
/usr/sbin/dnssec-dsfromkey  
/usr/sbin/dnssec-importkey  
/usr/sbin/dnssec-keyfromlabel  
/usr/sbin/dnssec-keygen  
/usr/sbin/dnssec-keymgr  
/usr/sbin/dnssec-revoke  
/usr/sbin/dnssec-settime  
/usr/sbin/dnssec-signzone  
/usr/sbin/dnssec-verify  
/usr/sbin/genrandom  
/usr/sbin/isc-hmac-fixup  
/usr/sbin/lwresd  
/usr/sbin/named  
/usr/sbin/named-checkconf  
/usr/sbin/named-checkzone  
/usr/sbin/named-compilezone  
/usr/sbin/named-journalprint  
/usr/sbin/nsec3hash  
/usr/sbin/rndc  
/usr/sbin/rndc-confgen  
/usr/sbin/tsig-keygen  
/usr/share/doc/bind-9.11.4  
/usr/share/doc/bind-9.11.4/Bv9ARM.ch01.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.ch02.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.ch03.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.ch04.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.ch05.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.ch06.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.ch07.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.ch08.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.ch09.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.ch10.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.ch11.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.ch12.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.ch13.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.html  
/usr/share/doc/bind-9.11.4/Bv9ARM.pdf  
/usr/share/doc/bind-9.11.4/CHANGES  
/usr/share/doc/bind-9.11.4/README  
/usr/share/doc/bind-9.11.4/isc-logo.pdf  
/usr/share/doc/bind-9.11.4/man.arpaname.html  
/usr/share/doc/bind-9.11.4/man.ddns-confgen.html  
/usr/share/doc/bind-9.11.4/man.delv.html  
/usr/share/doc/bind-9.11.4/man.dig.html  
/usr/share/doc/bind-9.11.4/man.dnssec-checkds.html  
/usr/share/doc/bind-9.11.4/man.dnssec-coverage.html  
/usr/share/doc/bind-9.11.4/man.dnssec-dsfromkey.html  
/usr/share/doc/bind-9.11.4/man.dnssec-importkey.html  
/usr/share/doc/bind-9.11.4/man.dnssec-keyfromlabel.html  
/usr/share/doc/bind-9.11.4/man.dnssec-keygen.html  
/usr/share/doc/bind-9.11.4/man.dnssec-keymgr.html  
/usr/share/doc/bind-9.11.4/man.dnssec-revoke.html  
/usr/share/doc/bind-9.11.4/man.dnssec-settime.html  
/usr/share/doc/bind-9.11.4/man.dnssec-signzone.html  
/usr/share/doc/bind-9.11.4/man.dnssec-verify.html  
/usr/share/doc/bind-9.11.4/man.dnstap-read.html  
/usr/share/doc/bind-9.11.4/man.genrandom.html  
/usr/share/doc/bind-9.11.4/man.host.html  
/usr/share/doc/bind-9.11.4/man.isc-hmac-fixup.html  
/usr/share/doc/bind-9.11.4/man.lwresd.html  
/usr/share/doc/bind-9.11.4/man.mdig.html  
/usr/share/doc/bind-9.11.4/man.named-checkconf.html  
/usr/share/doc/bind-9.11.4/man.named-checkzone.html  
/usr/share/doc/bind-9.11.4/man.named-journalprint.html  
/usr/share/doc/bind-9.11.4/man.named-nzd2nzf.html  
/usr/share/doc/bind-9.11.4/man.named-rrchecker.html  
/usr/share/doc/bind-9.11.4/man.named.conf.html  
/usr/share/doc/bind-9.11.4/man.named.html  
/usr/share/doc/bind-9.11.4/man.nsec3hash.html  
/usr/share/doc/bind-9.11.4/man.nslookup.html  
/usr/share/doc/bind-9.11.4/man.nsupdate.html  
/usr/share/doc/bind-9.11.4/man.pkcs11-destroy.html  
/usr/share/doc/bind-9.11.4/man.pkcs11-keygen.html  
/usr/share/doc/bind-9.11.4/man.pkcs11-list.html  
/usr/share/doc/bind-9.11.4/man.pkcs11-tokens.html  
/usr/share/doc/bind-9.11.4/man.rndc-confgen.html  
/usr/share/doc/bind-9.11.4/man.rndc.conf.html  
/usr/share/doc/bind-9.11.4/man.rndc.html  
/usr/share/doc/bind-9.11.4/named.conf.default  
/usr/share/doc/bind-9.11.4/notes.html  
/usr/share/doc/bind-9.11.4/notes.pdf  
/usr/share/doc/bind-9.11.4/sample  
/usr/share/doc/bind-9.11.4/sample/etc  
/usr/share/doc/bind-9.11.4/sample/etc/named.conf  
/usr/share/doc/bind-9.11.4/sample/etc/named.rfc1912.zones  
/usr/share/doc/bind-9.11.4/sample/var  
/usr/share/doc/bind-9.11.4/sample/var/named  
/usr/share/doc/bind-9.11.4/sample/var/named/data  
/usr/share/doc/bind-9.11.4/sample/var/named/my.external.zone.db  
/usr/share/doc/bind-9.11.4/sample/var/named/my.internal.zone.db  
/usr/share/doc/bind-9.11.4/sample/var/named/named.ca  
/usr/share/doc/bind-9.11.4/sample/var/named/named.empty  
/usr/share/doc/bind-9.11.4/sample/var/named/named.localhost  
/usr/share/doc/bind-9.11.4/sample/var/named/named.loopback  
/usr/share/doc/bind-9.11.4/sample/var/named/slaves  
/usr/share/doc/bind-9.11.4/sample/var/named/slaves/my.ddns.internal.zone.db  
/usr/share/doc/bind-9.11.4/sample/var/named/slaves/my.slave.internal.zone.db  
/usr/share/man/man1/arpaname.1.gz  
/usr/share/man/man1/named-rrchecker.1.gz  
/usr/share/man/man5/named.conf.5.gz  
/usr/share/man/man5/rndc.conf.5.gz  
/usr/share/man/man8/ddns-confgen.8.gz  
/usr/share/man/man8/dnssec-checkds.8.gz  
/usr/share/man/man8/dnssec-coverage.8.gz  
/usr/share/man/man8/dnssec-dsfromkey.8.gz  
/usr/share/man/man8/dnssec-importkey.8.gz  
/usr/share/man/man8/dnssec-keyfromlabel.8.gz  
/usr/share/man/man8/dnssec-keygen.8.gz  
/usr/share/man/man8/dnssec-keymgr.8.gz  
/usr/share/man/man8/dnssec-revoke.8.gz  
/usr/share/man/man8/dnssec-settime.8.gz  
/usr/share/man/man8/dnssec-signzone.8.gz  
/usr/share/man/man8/dnssec-verify.8.gz  
/usr/share/man/man8/genrandom.8.gz  
/usr/share/man/man8/isc-hmac-fixup.8.gz  
/usr/share/man/man8/lwresd.8.gz  
/usr/share/man/man8/named-checkconf.8.gz  
/usr/share/man/man8/named-checkzone.8.gz  
/usr/share/man/man8/named-compilezone.8.gz  
/usr/share/man/man8/named-journalprint.8.gz  
/usr/share/man/man8/named.8.gz  
/usr/share/man/man8/nsec3hash.8.gz  
/usr/share/man/man8/rndc-confgen.8.gz  
/usr/share/man/man8/rndc.8.gz  
/usr/share/man/man8/tsig-keygen.8.gz  
/var/log/named.log  
/var/named  
/var/named/data  
/var/named/dynamic  
/var/named/named.ca    13 个 rootnameserver 的信息 
/var/named/named.empty  
/var/named/named.localhost  
/var/named/named.loopback  
/var/named/slaves

1. /var/named/named.ca    13 个 rootnameserver 的信息 



### /var/named/named.ca
```bash
; <<>> DiG 9.11.3-RedHat-9.11.3-3.fc27 <<>> +bufsize=1200 +norec @a.root-servers.net
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46900
;; flags: qr aa; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 27

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1472
;; QUESTION SECTION:
;.                              IN      NS

;; ANSWER SECTION:
.                       518400  IN      NS      a.root-servers.net.
.                       518400  IN      NS      b.root-servers.net.
.                       518400  IN      NS      c.root-servers.net.
.                       518400  IN      NS      d.root-servers.net.
.                       518400  IN      NS      e.root-servers.net.
.                       518400  IN      NS      f.root-servers.net.
.                       518400  IN      NS      g.root-servers.net.
.                       518400  IN      NS      h.root-servers.net.
.                       518400  IN      NS      i.root-servers.net.
.                       518400  IN      NS      j.root-servers.net.
.                       518400  IN      NS      k.root-servers.net.
.                       518400  IN      NS      l.root-servers.net.
.                       518400  IN      NS      m.root-servers.net.

;; ADDITIONAL SECTION:
a.root-servers.net.     518400  IN      A       198.41.0.4
b.root-servers.net.     518400  IN      A       199.9.14.201
c.root-servers.net.     518400  IN      A       192.33.4.12
d.root-servers.net.     518400  IN      A       199.7.91.13
e.root-servers.net.     518400  IN      A       192.203.230.10
f.root-servers.net.     518400  IN      A       192.5.5.241
g.root-servers.net.     518400  IN      A       192.112.36.4
h.root-servers.net.     518400  IN      A       198.97.190.53
i.root-servers.net.     518400  IN      A       192.36.148.17
j.root-servers.net.     518400  IN      A       192.58.128.30
k.root-servers.net.     518400  IN      A       193.0.14.129
l.root-servers.net.     518400  IN      A       199.7.83.42
m.root-servers.net.     518400  IN      A       202.12.27.33
a.root-servers.net.     518400  IN      AAAA    2001:503:ba3e::2:30
b.root-servers.net.     518400  IN      AAAA    2001:500:200::b
c.root-servers.net.     518400  IN      AAAA    2001:500:2::c
d.root-servers.net.     518400  IN      AAAA    2001:500:2d::d
e.root-servers.net.     518400  IN      AAAA    2001:500:a8::e
f.root-servers.net.     518400  IN      AAAA    2001:500:2f::f
g.root-servers.net.     518400  IN      AAAA    2001:500:12::d0d
h.root-servers.net.     518400  IN      AAAA    2001:500:1::53
i.root-servers.net.     518400  IN      AAAA    2001:7fe::53
j.root-servers.net.     518400  IN      AAAA    2001:503:c27::2:30
k.root-servers.net.     518400  IN      AAAA    2001:7fd::1
l.root-servers.net.     518400  IN      AAAA    2001:500:9f::42
m.root-servers.net.     518400  IN      AAAA    2001:dc3::35

;; Query time: 24 msec
;; SERVER: 198.41.0.4#53(198.41.0.4)
;; WHEN: Thu Apr 05 15:57:34 CEST 2018
;; MSG SIZE  rcvd: 811
```



### name.conf
centos 安装后查看 `/usr/share/doc/bind*/sample/` 样例配置

```bash
#
# named.conf
#
# Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
# server as a caching only nameserver (as a localhost DNS resolver only).
#
# See /usr/share/doc/bind*/sample/ for example named configuration files.
#
# See the BIND Administrator's Reference Manual (ARM) for details about the
# configuration located in /usr/share/doc/bind-{version}/Bv9ARM.html

options {
	listen-on port 53 { 127.0.0.1; };
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	recursing-file  "/var/named/data/named.recursing";
	secroots-file   "/var/named/data/named.secroots";
	allow-query     { localhost; };

	/* 
	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
	   recursion. 
	 - If your recursive DNS server has a public IP address, you MUST enable access 
	   control to limit queries to your legitimate users. Failing to do so will
	   cause your server to become part of large scale DNS amplification 
	   attacks. Implementing BCP38 within your network would greatly
	   reduce such attack surface 
	*/
	recursion yes;

	dnssec-enable yes;
	dnssec-validation yes;

	/* Path to ISC DLV key */
	bindkeys-file "/etc/named.root.key";

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
	type hint;
	file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";


```

## dns record
[https://www.cloudflare.com/learning/dns/dns-records/](https://www.cloudflare.com/learning/dns/dns-records/)

authoritative dns server  -> DNS records(zone files) -> TTL



### `A` record
address record

[https://www.cloudflare.com/learning/dns/dns-records/dns-a-record/](https://www.cloudflare.com/learning/dns/dns-records/dns-a-record/)


### `CNAME` record
canonical name

[https://www.cloudflare.com/learning/dns/dns-records/dns-cname-record/](https://www.cloudflare.com/learning/dns/dns-records/dns-cname-record/)

指向另一个域名

如 blog.example.com 有一个 CNAME record 是 example.com。

当一个 dns server 命中 blog.example.com 的 dns record 时，实际上会触发另外一个 example.com 的 dns lookup。返回 example.com 的 A record(ip 地址)。

blog.example.com CNAME example.com
shop.example.com CNAME example.com

如果 ip 变化了，只需要修改 example.com 的 A record


### `NS` record
nameserver

[https://www.cloudflare.com/learning/dns/dns-records/dns-ns-record/](https://www.cloudflare.com/learning/dns/dns-records/dns-ns-record/)

 存储了一个 domain 所有的 DNS records，包括 A record, MX record, CNAME record. 多个 NS 增加可用性。

```bash
dog -t NS taobao.com
NS taobao.com. 1s   "ns5.taobao.com."
NS taobao.com. 1s   "ns4.taobao.com."
NS taobao.com. 1s   "ns6.taobao.com."
NS taobao.com. 1s   "ns7.taobao.com."
```

```bash
dog -t NS ns5.taobao.com.
SOA taobao.com. 20m00s A "ns4.taobao.com." "hostmaster.alibabadns.com." 2018011109 1h00m00s 20m00s 1h00m00s 6m00s
```



### SOA record
[https://www.cloudflare.com/learning/dns/dns-records/dns-soa-record/](https://www.cloudflare.com/learning/dns/dns-records/dns-soa-record/)

> store `admin info` about a domain.

start of authority

DNS SOA record 存储一个 domain 或者 zone 的重要信息。包括：

email address of the administrator

when the domain was last updated

how long the server should wait between refreshes.

[https://www.okta.com/identity-101/soa-record/#:~:text=A%20DNS%20SOA%20record%20contains,party%20responsible%20for%20the%20domain.](https://www.okta.com/identity-101/soa-record/#:~:text=A%20DNS%20SOA%20record%20contains,party%20responsible%20for%20the%20domain.)



```bash
 dig soa baidu.com

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.15 <<>> soa baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44818
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;baidu.com.                     IN      SOA

;; ANSWER SECTION:
baidu.com.              2849    IN      SOA     dns.baidu.com. sa.baidu.com. 2012147685 300 300 2592000 7200

;; Query time: 30 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: 一 12月 25 14:24:35 CST 2023
;; MSG SIZE  rcvd: 81
```



> ;; ANSWER SECTION:
>

baidu.com.              2849    IN      SOA     dns.baidu.com. sa.baidu.com. 2012147685 300 300 2592000 7200

MNAME  <u>dns.baidu.com. </u>  _**primary server for this zone  **__**<font style="color:#DF2A3F;">zone's master server</font>**_

RNAME <u>sa.baidu.com. </u>   _**administrator's email address，但是没有 <font style="color:#DF2A3F;">@</font>**_

SERIAL <u>2012147685</u> _**a number representation of changes，每次修改序列号加 1**_

REFRESH <u>300</u>  The time in seconds secondary servers should wait before asking the primary server for a new SOA record indicating changes.

RETRY <u>300</u> How long a server should wait after a failed refresh before asking again.

EXPIRE  <u>2592000</u> The upper time limit before a zone isn't considered authoritative.

MINIMUM <u>7200</u> A measurement of "time to live" or how long a resolver should wait.



### `PTR` record
[https://www.cloudflare.com/learning/dns/dns-records/dns-ptr-record/](https://www.cloudflare.com/learning/dns/dns-records/dns-ptr-record/)

> pointer(PTR) 通过 ip 找域名


## dns server type
[https://www.cloudflare.com/learning/dns/dns-server-types/](https://www.cloudflare.com/learning/dns/dns-server-types/)

### recursive resolvers
DNS recursor -- first stop 第一站，client 向 recursive resolver 发起一个递归查询。

收到查询请求后，要么直接返回 cached data，要么向 root nameserver 发起查询，root nameserver 返回对应的 TLD nameserver。接着向 TLD nameserver 发起查询，依次类推，最终会向 authoritative nameserver 发起请求，返回对应的 domain 的 ip 地址。同时也会将结果缓存下来。

### root nameservers
所有的 recursive resolver 都会知道 root nameservers 的 ip。

所有 recursive resolver 查询第一站就是 root nameserver, 树形结构。

### TLD nameservers
记录 顶级域名如 .com 名字信息的服务器。

### authoritative nameservers
resolver last stop。有具体域名的 ip 信息。如 google.com。可以向 recursive resolver 提供对应的 ip（A record）地址。如果域名有 CNAME record(alias) 向 recursive resolver 提供 alias 域名，这时 recursive resolver 会发起一个全新的 DNS lookup。


### client(stub resolver)




## nsupdate  动态更新 DNS 命令行工具
1. [https://dnns.no/dynamic-dns-with-bind-and-nsupdate.html](https://dnns.no/dynamic-dns-with-bind-and-nsupdate.html)
2. [https://bind9.readthedocs.io/en/v9.18.21/chapter6.html#dynamic-update](https://bind9.readthedocs.io/en/v9.18.21/chapter6.html#dynamic-update)



### 使用 ddns-confgen 生成 key

```bash
ddns-confgen -k key1 

# To activate this key, place the following in named.conf, and
# in a separate keyfile on the system or systems from which nsupdate
# will be run:
key "key1" {
        algorithm hmac-sha256;
        secret "N0c+DmoYWriFeN8SAwGg+i+E+CS4jMRplzs8+fGlLAQ=";
};

# Then, in the "zone" statement for each zone you wish to dynamically
# update, place an "update-policy" statement granting update permission
# to this key.  For example, the following statement grants this key
# permission to update any name within the zone:
update-policy {
        grant key1 zonesub ANY;
};

# After the keyfile has been placed, the following command will
# execute nsupdate using this key:
nsupdate -k <keyfile>
```

### 配置 named.conf 的 view
```bash
view "internal" {
        match-clients { 10.101.0.0/24; 10.101.9.11/32; key key1;  };
        key "key1" {
        algorithm hmac-sha256;
        secret "N0c+DmoYWriFeN8SAwGg+i+E+CS4jMRplzs8+fGlLAQ=";
       };

        zone "myexample.com" IN {
        type master;
        file "myexample.com.zone";
        update-policy {
           grant key1  zonesub ANY;
        };
        notify yes;
        also-notify { 10.101.0.182; };
        allow-transfer { 10.101.0.182; };

        };

        zone "." IN {
        type hint;
        file "named.ca";
        };

        include "/etc/named.rfc1912.zones";
};
```

## 如何实现动态更新 Dns

根据实际的域名信息变化，生成对应的 `nsupdate` 命令，再执行即可。


# References
1. [https://www.cloudflare.com/learning/dns/glossary/dns-zone/](https://www.cloudflare.com/learning/dns/glossary/dns-zone/)
2. [https://www.cloudflare.com/learning/dns/dns-records/](https://www.cloudflare.com/learning/dns/dns-records/)
3. [https://root-servers.org/](https://root-servers.org/)
4. [https://man.openbsd.org/resolv.conf.5](https://man.openbsd.org/resolv.conf.5)
5. [https://www.cloudflare.com/learning/cdn/glossary/anycast-network/](https://www.cloudflare.com/learning/cdn/glossary/anycast-network/)
6. [https://serverfault.com/questions/220775/what-does-the-in-mean-in-a-zone-file](https://serverfault.com/questions/220775/what-does-the-in-mean-in-a-zone-file)
7. [https://en.wikipedia.org/wiki/TSIG](https://en.wikipedia.org/wiki/TSIG) 更新 dns 需要
8. [https://linux.die.net/man/8/ddns-confgen](https://linux.die.net/man/8/ddns-confgen)
9. [https://www.acunetix.com/blog/articles/dns-zone-transfers-axfr/](https://www.acunetix.com/blog/articles/dns-zone-transfers-axfr/)
10. [https://datatracker.ietf.org/doc/html/rfc2136](https://datatracker.ietf.org/doc/html/rfc2136) Dynamic Updates in the Domain Name System
11. [https://www.okta.com/identity-101/hmac/#:~:text=Hash%2Dbased%20message%20authentication%20code,use%20signatures%20and%20asymmetric%20cryptography.](https://www.okta.com/identity-101/hmac/#:~:text=Hash%2Dbased%20message%20authentication%20code,use%20signatures%20and%20asymmetric%20cryptography.)
12. [https://bind9.readthedocs.io/en/v9.18.21/chapter1.html#referral](https://bind9.readthedocs.io/en/v9.18.21/chapter1.html#referral)
13. [https://dnns.no/dynamic-dns-with-bind-and-nsupdate.html](https://dnns.no/dynamic-dns-with-bind-and-nsupdate.html)
14. [https://bind9.readthedocs.io/en/v9.18.21/manpages.html#man-dig](https://bind9.readthedocs.io/en/v9.18.21/manpages.html#man-dig)
15. [https://netdns2.com/documentation/request-signing-sig0-and-tsig/](https://netdns2.com/documentation/request-signing-sig0-and-tsig/)
16. [https://bind9.readthedocs.io/en/v9.18.21/reference.html#statements](https://bind9.readthedocs.io/en/v9.18.21/reference.html#statements)
17. [https://www.cloudflare.com/dns/dnssec/how-dnssec-works/](https://www.cloudflare.com/dns/dnssec/how-dnssec-works/)    
18. [https://blog.stoplight.io/api-keys-best-practices-to-authenticate-apis](https://blog.stoplight.io/api-keys-best-practices-to-authenticate-apis)
19. [https://bind9.readthedocs.io/en/v9.18.21/chapter7.html#dynamic-update-security](https://bind9.readthedocs.io/en/v9.18.21/chapter7.html#dynamic-update-security)
20. [https://www.rtfm-sarl.ch/articles/using-nsupdate.html](https://www.rtfm-sarl.ch/articles/using-nsupdate.html)
21. [https://dnns.no/dynamic-dns-with-bind-and-nsupdate.html](https://dnns.no/dynamic-dns-with-bind-and-nsupdate.html) 强烈推荐
22. [https://bind9.readthedocs.io/en/v9.18.21/manpages.html#nsupdate-dynamic-dns-update-utility](https://bind9.readthedocs.io/en/v9.18.21/manpages.html#nsupdate-dynamic-dns-update-utility)
23. [https://mp.weixin.qq.com/s/DLKzJ_Paeh52g2SoQkhSVQ](https://mp.weixin.qq.com/s/DLKzJ_Paeh52g2SoQkhSVQ)
24. [http://caunter.ca/nsupdate.txt](http://caunter.ca/nsupdate.txt)
25. [https://datatracker.ietf.org/doc/html/rfc2845](https://datatracker.ietf.org/doc/html/rfc2845)
26. [https://nsrc.org/workshops/2011/pacnog9-dns-ops/raw-attachment/wiki/Agenda/TSIG-17022011.pdf](https://nsrc.org/workshops/2011/pacnog9-dns-ops/raw-attachment/wiki/Agenda/TSIG-17022011.pdf)
27. [https://bind9.readthedocs.io/en/v9.18.21/reference.html#address-match-lists](https://bind9.readthedocs.io/en/v9.18.21/reference.html#address-match-lists)  非常重要 key 也可以匹配
28. [https://serverfault.com/questions/592492/updates-to-a-bind-dynamic-zone-that-is-shared-between-views-delayed](https://serverfault.com/questions/592492/updates-to-a-bind-dynamic-zone-that-is-shared-between-views-delayed)    match-clients 不仅仅是 ip 地址
29. [https://datatracker.ietf.org/doc/html/rfc2845](https://datatracker.ietf.org/doc/html/rfc2845)        Secret Key Transaction Authentication for DNS (TSIG)
30. [https://hostman.com/tutorials/setting-up-a-bind-dns-server/](https://hostman.com/tutorials/setting-up-a-bind-dns-server/) 配置bind9
31. [https://www.unixmen.com/dns-server-setup-using-bind-9-on-centos-7-linux/](https://www.unixmen.com/dns-server-setup-using-bind-9-on-centos-7-linux/) 配置
32. [https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-18-04)

