---
title: Docker 与 Linux Namespace && Cgroups
copyright: true
date: 2019-06-30 13:40:28
tags: Docker
categories: Docker
---

# Docker 与 Linux Namespace && Cgroups

## 什么是Docker?

不得不说这几年技术领域最火的就是“容器”这个词了。而谈到容器，大家的第一反应就是 Docker ,Docker 已经再很多人心中成了容器的代名词。那么Docker到底是什么？Docker能为我们带来什么？

官网上用来解释Docker的一句话:`Docker is a platform for developers and sysadmins to develop, deploy, and run applications with containers`。Docker 就是一个开源的工具，将我们的应用打包成标准的镜像格式，并且以容器的方式运行。容器化的越来越流行，带给我们的优势也是非常多：

1. 灵活性：再复杂的应用都可以被容器化；
2. 轻量级：容器利用共享的是主机内核；
3. 即时性：可以随时部署更新和升级；
4. 通用性：一次构建，到处运行；
5. 伸缩性：控制容器副本数量来任意伸缩；
<!--more-->
## Docker 与虚拟机的比较

容器与容器之间是共享Kernel的，各容器直接互相隔离。它只运行一个独立的进程，没有其它的执行进程，也不需要占用其它额外的资源。

虚拟机运行的是一个独立的完整的系统，占用的资源也要比独立的应用需要的多。他们两者区别在于虚拟机管理程序对整个设备进行抽象处理，而容器只是对操作系统内核进行抽象处理。下面这张图可以对两者有个认知了解：

![2019-06-30-15-12-00](/images/qiniu/2019-06-30-15-12-00.png)

## Linux Namespace

我们经常听到，Docker其实并不是单独创造的一个技术，在早期，Docker其实就是基于Linux上的LXC(Linux Container)项目来创建单个应用程序的容器，目前Docker使用libcontainer来直接操作核心namespace和cgoup。这里我们了解下Linux Namesapce。Linux namespace是Kernel的功能，主要用来隔离一系列资源，目前Linux有6种不同类型的Namespace：

1. Mount Namespace, CLONE_NEWNS, 用来隔离nodename和domainname;
2. UTS Namespace, CLONE_NEWUTS, 用来隔离 System V IPC 和 POSIX message queues;
3. IPC Namespace, CLONE_NEWIPC, 用来隔离进行ID;
4. PID Namespace, CLONE_NEWPID, 用来隔离各个进程看到的挂载点视图;
5. Network Namespace, CLONE_NEWNET, 用来隔离网络设备、IP 地址端口等网络栈的 Namespace;
6. User Namespace, CLONE_NEWUSER, 用来隔离用户的用户组ID;

## Linux Cgroups

Linux Cgroups (Control Groups) 提供了一组进程及将来子进程的资源限制、控制和统计的能力，资源包括CPU、内存、存储、网络等。通过Cgroups,可以方便地限制某个进程的资源占用，并且可以实时地监控进程的监控和统计信息。Cgroups的三个组件，

1. cgroup
2. subsystem
3. hierarchy

## libcontainer

libcontainer 是Docker开源的一个项目，目前runC的实现也已经有原来的LXC变为libcontainer。官网对libcontainer的解释：[libcontainer provides a native Go implementation for creating containers with namespaces, cgroups, capabilities, and filesystem access controls. It allows you to manage the lifecycle of the container performing additional operations after the container is created](https://docs.docker.com/glossary/?term=libcontainer)


