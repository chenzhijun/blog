---
title: 修改 Linux Core 文件目录
copyright: true
date: 2019-05-08 20:00:11
tags: Linux
categories: Linux
---

# Linux Core 文件

一次线上的经历，一台现在主机突然磁盘根目录占到98%。我们其实是挂了数据盘的，不知道为啥突然会
在根目录下磁盘空空间不足报警。上到主机一看，原来是某个应用打了dump了个core文件。文件如：core.xxxx

用gdb调试了一下：

`gdb core.14321`

显示出是哪个进程dump的core文件。

之后我们就开始分析core文件怎么避免让根目录占满。

首先用`ulimit -a`查看到core文件的大小。然后用`ulimit -c`设置大小，单位是block，
1block=512bytes

然后修改core文件到数据盘的目录：

`echo /data/coredump/core.%e.%p> /proc/sys/kernel/core_pattern`