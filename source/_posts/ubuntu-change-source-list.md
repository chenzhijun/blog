---
title: ubuntu 怎么修改源？
copyright: true
date: 2018-05-02 17:52:56
tags: ubuntu
categories: ubuntu
---

# ubuntu 怎么修改源

修改ubuntu的源为国内的源，修改之前首先备份：

```shell
cp /etc/apt/sources.list /etc/apt/sources.list.backup
```

修改源列表，选择的是国内的阿里源,`vi /etc/apt/sources.list`,清空里面内容：`dG`,使用命令的时候要回到第一行：`gg`,之后将下面的内容复制到文件中：

```
deb-src http://archive.ubuntu.com/ubuntu xenial main restricted #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties
deb http://archive.canonical.com/ubuntu xenial partner
deb-src http://archive.canonical.com/ubuntu xenial partner
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb [arch=amd64] https://download.daocloud.io/docker/linux/ubuntu xenial stable
# deb-src [arch=amd64] https://download.daocloud.io/docker/linux/ubuntu xenial stable
deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse

```