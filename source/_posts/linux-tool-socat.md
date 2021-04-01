---
title: Linux 工具-socat
copyright: true
date: 2019-06-30 16:20:53
tags: tools
categories: Linux
---

# Linux 工具-socat

socat是一个多功能的网络工具，官网：[http://www.dest-unreach.org/socat/](http://www.dest-unreach.org/socat/)

## 安装socat

安装方式很简单：`yum install -y socat`,就可以了，当然如果是ubuntu的机器就是用`apt`

## 使用socat

公司内部的网络限制比较严格，只有一些常用的端口能正常访问，因此调试的时候非常麻烦，比如你的应用程序端口是19995，但是公司只能是8080来访问，这个时候怎么办？使用nginx或haproxy当然可以，但是麻烦啊，配置搞一堆。。但是使用socat就很方便了:`socat TCP4-LISTEN:{port1},reuseaddr,fork TCP4:{ip:port2}` ，比如你有三台机器A(127.0.0.1)；B(127.0.0.2）；C（192.168.1.1）。B能访问A的8080端口，但是不能访问C的9090端口，而服务又监听的是C的9090端口，A能访问C的9090端口。所以很当然会想到B-->A:8080-->C:9090。也就是在A做一层反向代理。socat就是这样的。在A上我们执行：`socat TCP4-LISTEN:8080,reuseaddr,fork TCP4:192.168.1.1:9090`。然后B就访问A:8080,就能访问到C的9090端口了。