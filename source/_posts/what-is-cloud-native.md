---
title: 什么是云原生？
copyright: true
date: 2023-02-10 14:27:45
tags: Kubernetes
categories: Kubernetes
---

# 什么是云原生？

云原生是一种基于容器、微服务和自动化运维的软件开发和部署方法。

云原生分为 3 种核心技术和 2 个核心理念：

3 种核心技术：分别是容器、微服务、服务网格；
2 个核心理念：分别指不可变基础设施和声明式 API。
<!--more-->
容器：容器是云原生应用程序中最小的计算单元。它们是将微服务代码和其他必需文件打包在云原生系统中的软件组件。通过容器化微服务，云原生应用程序独立于底层操作系统和硬件运行。这意味着软件开发人员可以在本地、云基础设施或混合云上部署云原生应用程序。 开发人员使用容器将微服务与其各自的依赖项（例如主应用程序运行所需的资源文件、库和脚本）打包。

微服务：微服务是小型的独立软件组件，它们作为完整的云原生软件共同运行。每个微服务都侧重于一个小而具体的问题。微服务是松散耦合的，这意味着它们是相互通信的独立软件组件。开发人员通过处理单个微服务来更改应用程序。这样，即使一个微服务出现故障，应用程序仍能继续运行。

服务网格：服务网格是云基础设施中的一个软件层，用于管理多个微服务之间的通信。开发人员使用服务网格来引入其他功能，而无需在应用程序中编写新代码。

不可变基础设施：不可变基础设施意味着用于托管云原生应用程序的服务器在部署后保持不变。如果应用程序需要更多计算资源，则会丢弃旧服务器，并将应用程序移至新的高性能服务器。通过避免手动升级，不可变基础设施使云原生部署成为一个可预测的过程。 

声明式 API：应用程序编程接口（API）是两个或多个软件程序用来交换信息的方法。云原生系统使用 API 将松散耦合的微服务整合在一起。API 会告诉您微服务想要什么数据以及它能给您带来什么结果，而不是指定实现结果的步骤。

云计算是云供应商按需提供的资源、基础设施和工具。而云原生是一种使用云计算模型构建和运行软件程序的方法。