---
title: golang编译成Linux环境下的二进制文件
copyright: true
date: 2019-03-31 21:38:08
tags: golang
categories: golang
---

# golang编译成Linux环境下的二进制文件

最近需要将golang项目编译成Linux下可执行的脚本，手中只有windows笔记本，服务器上又各种网络限制。
于是干脆就再本地打成Linux二进制文件，然后传到服务器直接启动。

在本地上（win10）打开控制台，然后进入到项目根路径。之后设置当前几个值：

```
SET CGO_ENABLED=0
SET GOOS=linux 
SET GOARCH=amd64

```

最后使用`go build .`就可以了。 也算是异常简单。

如果在`*nix`平台:

`export CGO_ENABLED=0 && export GOOS=linux && export GOARCH=amd64 && go build .`