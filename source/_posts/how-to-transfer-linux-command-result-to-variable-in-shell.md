---
title: 怎样将shell中命令执行的结果赋值给变量
copyright: true
date: 2019-03-31 21:51:57
tags: shell
categories: Linux
---

# 怎样将shell中命令执行的结果赋值给变量

其实这个比较简单。只需要将命令使用反单引号起来就可以了。

```shell

srvname=`docker inspect 1swd3|grep "name"|awk -F "," '{print $1}'|awk '{print $NF}'`

```
