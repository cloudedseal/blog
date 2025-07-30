---
date: '2025-07-31T06:25:50+08:00'
draft: false
title: 'Bing9 Reverse Dns'
---

## 配置
网段 x.y 配置反向的 DNS

zone _**<font style="color:#DF2A3F;">y.x.</font>**_**in-addr.arpa**


```bash
view "MASTER" {
        match-clients { 10.10.0.111/32; 10.10.9.11/32; key key1;  };
        key "key1" {
        algorithm hmac-sha256;
        secret "N0c+DmoYWriFeN8SAwGg+i+E+CS4jMRplzs8+fGlLAQ=";
       };
        zone "10.10.in-addr.arpa" IN {
        type master;
        file "db.10.10";
        update-policy {
         grant key1 zonesub ANY;
       };
       notify yes;
       also-notify { 10.10.0.182; };
        allow-transfer { 10.10.0.182; };
       };
        zone "." IN {
        type hint;
        file "named.ca";
        };
        include "/etc/named.rfc1912.zones";
};
```



## db.10.10 zone 文件

```bash
$ORIGIN .
$TTL 60 ; 1 minute
10.10.in-addr.arpa      IN SOA  10.10.in-addr.arpa. root.myexample.com. (
                                2023111495 ; serial
                                604800     ; refresh (1 week)
                                86400      ; retry (1 day)
                                2419200    ; expire (4 weeks)
                                604800     ; minimum (1 week)
                                )
                        NS      ns1.myexample.com.
                        NS      ns2.myexample.com.
$ORIGIN 10.10.in-addr.arpa.
$TTL 30 ; 30 seconds
34.12                   PTR     shogun14.myexample.com.
```



## nsupdate 命令

### 添加
```bash
nsupdate -y hmac-sha256:key1:N0c+DmoYWriFeN8SAwGg+i+E+CS4jMRplzs8+fGlLAQ= <<"EOF"
server 10.10.0.181 53

update add 34.12.10.10.in-addr.arpa 30 PTR shogun14.myexample.com
send
EOF

```



### 删除
```bash
nsupdate -y hmac-sha256:key1:N0c+DmoYWriFeN8SAwGg+i+E+CS4jMRplzs8+fGlLAQ= <<"EOF"
server 10.10.0.181 53

update del 34.12.10.10.in-addr.arpa 30 PTR
send
EOF
```



## References
1. [https://www.isc.org/docs/ReverseDNSpresentation.pdf](https://www.isc.org/docs/ReverseDNSpresentation.pdf)
2. [https://www.interserver.net/tips/kb/bind-reverse-dns-example-setup/](https://www.interserver.net/tips/kb/bind-reverse-dns-example-setup/)
3. [https://www.apnic.net/about-apnic/corporate-documents/documents/resource-guidelines/reverse-zones/](https://www.apnic.net/about-apnic/corporate-documents/documents/resource-guidelines/reverse-zones/)


