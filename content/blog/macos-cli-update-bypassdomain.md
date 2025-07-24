---
date: '2025-07-24T10:56:10+08:00'
draft: false
title: 'Macos Cli Update Bypassdomain'
---

## 起因

> 在 GUI 配置 bypassdomain 后，有时候会丢失，频繁配置太麻烦了。

## 配置存储在何处?

`/Library/Preferences/SystemConfiguration/preferences.plist`

## 使用命令行修改

### 查看 network services

```bash
networksetup -listallnetworkservices
An asterisk (*) denotes that a network service is disabled.
AX88179A
Wi-Fi
```

### 查看当前 bypassdomain 配置

```bash
networksetup -getproxybypassdomains AX88179A
127.0.0.1
192.168.0.0/16
10.0.0.0/8
172.16.0.0/12
172.29.0.0/16
localhost
*.local
*.crashlytics.com
<local>
```

### 追加新的 bypassdomain

```bash
sudo networksetup -setproxybypassdomains AX88179A "$(sudo networksetup -getproxybypassdomains AX88179A | tr '\n' ',' | xargs) *.example.com"
```

1. 使用 `tr '\n' ','` 转换成 CSV 形式。AI 一直坚持使用空格分割，是错误的。直到看到配置文件不对。🤣


## References

1. [QWEN](https://chat.qwen.ai/c/c784ac2e-455b-4caa-917b-cde61518837b)