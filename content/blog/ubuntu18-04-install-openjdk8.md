---
title: ubuntu18.04-install-openjdk8
date: 2019-05-25T14:33:35+08:00
tags: ["openjdk"]
---
# 添加 openjdk8 的第三方源

>sudo add-apt-repository ppa:openjdk-r/ppa
ppa (Personal Package Achives)

# 执行更新

>apt-get update

# 安装 openjdk8

>sudo apt-get install openjdk-8-jdk

# 选择版本

>sudo update-alternatives  - -config java

# 确认安装成功

>java -version
