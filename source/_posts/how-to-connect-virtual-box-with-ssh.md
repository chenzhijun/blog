---
title: 怎样用SSH连接VirtualBox
date: 2017-04-26 23:52:47
tags:
	- Linux
categories: Linux
---

### 用 iterm2 ssh连接virtual box中运行的centos minimal版本


#### 基础准备

* 安装好虚拟机vb
* centos官网下载minimal版本，也可以下载dvd版本之后只安装minimal
* iterm2

安装过程很简单，直接点击vb的new，之后按照提示一步一步点下去就可以了。
[给个安装教程网址](http://www.jianshu.com/p/2a853d569228)

#### 特殊注意点

如果选择`host-only adapter`的时候出现无法确认，或者无法选择的情况。这种情况下是因为虚拟机本身没有开一个host-only adapter。可以打开vb的系统设置，然后找到网络(network),选择Host-Only Networks。新建一个就可以了
![](/images/vb-host-only-sys.png)

有问题欢迎留言，一起交流学习。 

