---
title: kubernetes 基础 一（简介与核心组件介绍）
copyright: true
date: 2021-01-17 22:51:32
tags: Kubernetes
categories: Kubernetes
---

# kubernetes 基础 一（简介与核心组件介绍）

## Kubernetes 是什么？

Kubernetes 是一个用于编排容器化服务的的开源平台。我们通常会将 Kubernetes 简称为 K8s。Kubernetes 通常也被大家当做容器编排引擎，可以这样说，目前市面上很多的 PaaS 平台基本都是使用 k8s 作为其底层支持。

<!--more--> 

## 核心功能

如果了解到 kubernetes 是什么，那么 kubernetes 为什么我们要使用了？它提供了我们什么能力而让其这么火？

### 应用编排与服务发现

kubernetes 能够帮助我们管理我们的业务应用系统，并且在内部可以基于dns 和 ip 来进行通信访问，对外可以提供 service 和 loadbalance 两种方式提供服务访问。

### 储存管理

kubernetes 提供多种方式来进行数据的持久化管理，比如本地卷，ceph，第三方公有云的存储等

### 自动发布与回滚

kubernetes 可以工具你配置的应用的desired 来以可控的方式控制应用的发布与回滚

### 应用自愈

kubernetes 可以在服务状态不对时，进行服务自愈。比如健康检查

### 秘钥与配置管理

可以在应用无需重启就能更新服务的 secret 和配置 config。这个功能其实可以做比如动态更新配置，多环境配置等。

## 核心控制平面组件

![2020-06-07-17-19-38](/images/qiniu/2020-06-07-17-19-38.png)

kubernetes 集群中机器分为两个类型，控制节点和工作节点；kubernetes 组件又分为控制面组件和节点组件。比如控制平面组件为：kube-apserver,kube-scheduler,kube-controller-manager，ETCD；这些控制平面的组件只会运行在 master 节点上。而节点组件：kubelet，kube-proxy 则会运行在集群所有节点上（包括 master 和 worker）。接下来我么来了解下每个组件的功能是做什么的。

### kube-apiserver

kube-apiserver 是控制平面的组件。主要用来暴露 kubernetes 的 api。kube-apiserver 设计成可水平扩展的应用，用一个 loadbalance 就可以对其所有 apiserver 的实例进行负载。

### kube-scheduler

主要用来监听所有没有调度主机的新创建的 pod，并且将 pod 调度到主机上。scheduler 调度规则包含了很多因素：独立和集群需要的资源，硬件、软件、策略约束，亲和性和非亲和性，数据约束，业务干扰，调度时间等

### kube-controller-manager

kubernetes 的主要控制器，通过 apiserver 来监控整个集群的状态，并且确保集群处于预期的工作状态。它由一系列独立的 controller 组成。为了减少复杂性，他们封装进了一个二进制文件中，并且运行一个进程。

这些 controller 包括：Node controller，Replication controller，Endpoints controller，Service Account & Token controllers等等。

### ETCD

etcd 本身是一个高可用的键值数据库，kubernetes 用其来进行集群的数据存储。

## 核心节点组件
### kubelet

在集群中的每个主机上都会运行的一个 agent，确保容器在 pod 中运行。kubelet 通过PodSpec来管理容器，并且确保他们运行正常。注意：如果是主机原来的容器，kubelet 并不会进行管理。

### kube-proxy

kube-proxy 是每个主机上都会有的一个网络代理，是 service 的主要实现方式。kube-proxy 包含主机上的网络规则，这些规则在集群中定了集群内部的访问和外部的网络会话访问方式。

## 集群插件

集群插件都是属于集群级别的组件，以deployment，daemonset等方式运行在 kube-system 的命名空间下，用来实现集群级别的一些功能。

### DNS 插件

DNS 是集群中的必须安装的插件，在集群中很多功能都会依赖 dns 。kubernetes 创建的容器会自动包含这个 dns 服务器在容器的 dns 搜索域中。
