---
title: Kubernetes 第一天
copyright: true
date: 2019-04-03 21:10:22
tags: Kubernetes
categories: Kubernetes
---

# Kubernetes day1

## Kubernetes是什么？

Kubernetes简称k8s。最直观的解释它是用将资源进行整合，将应用与底层隔离。因此，搞业务开发的，专心搞好业务逻辑。搞底层机器，网络资源的安心搞好底层网络资源。而k8s就是这中间的一层，承上启下。也就是传说中的PaaS。现在可以说没有那个Paas或者Caas不是基于k8s搞的。不过我觉得理解的不深，k8s的作用其实还有很多，如果你用过docker或者直接开发业务就会有比较深的感知。

## Kubernetes 关键组件

安装一个K8S集群需要哪些组件了？一个完整的小集群里面，需要有一个master，一个node，两台机器。master和node上分别有哪些组件了？master上有kube-apiserver,kube-scheduler,kube-controller-manager; node上有kubelet,kube-proxy,docker;另外需要在master上安装一个etcd数据库。如果是非二进制安装，master和node上都需要有kubeadm。安装完这些之后需要在master节点上使用'kubectl apply -f [kubelet-network].yaml'也就是如果你使用flannel网络，就可能需要安装flannel的网络插件。这个插件是已pod的方式创建的。

## 组件说明

### Etcd

键值数据库，这个没有什么特别好说的。要在安装k8s集群前先启动，保存k8s所有资源对象数据

### kube-apiserver

集群控制的入口进程，提供HTTP Rest接口的关键服务进程，是k8s里面所有资源操作的唯一入口。

### kube-controller-manager

k8s所有资源对象的自动化控制中心

### kube-scheduler

k8s的资源调度进程

### kubelet

负责pod对应的容器的创建，启停等任务，同时与Maser节点密切协作，实现集群管理的基本功能

### kube-proxy

实现kubernetes Service的通信与负载均衡机制的重要组件

