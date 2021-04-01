---
title: Docker 容器
date: 2017-05-13 14:40:24
tags:
	- Docker
categories: Docker
---

## Docker 容器

容器是 Docker 的另一个核心概念

> 容器是镜像的一个运行实例，所不同的是，它带有额外的可写文件层

### 1,创建容器
`docker create -it centos:latest` 新建的容器处于停止状态，可以用 `docker start` 命令来启动它。

### 2,新建并启动容器
`docker run centos /bin/echo 'Hello world'` ，输出了Hello world,之后就没了,使用 `docker ps` 看不到容器运行。
<!--more-->
Docker 后台的操作分解：
```
	* 检查本地是否存在指定的镜像，不存在就从共有仓库下载
	* 利用镜像创建并启动一个容器
	* 分配一个文件系统，并在只读的及各项层外面挂载一层可读写层
	* 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
	* 从地址池配置一个IP地址给容器
	* 执行用户指定的应用程序
	* 执行完毕后容器被终止
```

`docker run -ti ubuntu:latest /bin/bash`  ，在容器内部运行ps命令，可以看到只有bash应用 
```
	-t 让Docker分配一个伪终端并绑定到容器的标准输入上
	-i 则让让其的标准输入保持打开
```
用exit退出后，容器也停止运行，处于终止状态。

#### 2.1,守护态运行
很多时候需要让Docker容器在后台以守护态形式运行，用户可以通过添加`-d`参数来实现。
`docker run -d ubuntu /bin/sh -c "while true;do echo hello world;sleep 1;done"`,会生成一个id。
之后用`docker logs id`就能看到记录日志。

### 2.2,重启容器
`docker restart container`，会将一个运行态的容器终止，然后重新启动它。

### 3,终止容器
`docker stop [-t|--time[=10]] container`，它会先向容器发送SIGTERM信号，等待-t秒后（默认10秒），再发送SIGKILL信号

### 4,进入容器
使用`-d`参数后，容器进入后台运行。有时候需要计入到容器里面进行操作，可以使用`docker attach ` ,`docker exec `等命令。

a) attach 进入到容器
`docker attach 65bde56eaf56-容器id`,`attach` 命令进入到容器里面，不过要首先启动容器，另外当多个窗口同时attach到同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时，其它窗口也无法执行操作。

b) exec 进入到容器
docker 1.3之后，提供了一个更方便的工具exec，可以直接在容器内运行命令。
`docker exec -ti 65bde56eaf56[|container-name] /bin/bash[|bash|other-command]`可以很方便的进入到容器里面

### 5,删除容器
`docker rm [OPTIONS] container-id1[|container-name1] container-id2[|container-name2]` ,删除处于停止运行的容器。
```
	* -f, --force=false 强行终止并删除一个运行中的容器
	* -l, --link=false 删除容器的连接，但保留容器
	* -v, --volumes=fasle 删除容器挂载的数据卷
```

### 6，导入和导出容器
a) 导出容器
导出容器是指导出一个已经创建的容器到一个文件，不管此时这个容器是否处于运行状态，可以使用docker export 命令`dcoker export container`.
`docker export container-id[|container-name] > test_export.tar`;
可以将导出的文件传输到其它机器上，在其它机器上通过导入命令实现容器的迁移。

b) 导入容器
可以将导出的文件又导入，成为镜像
`cat test_export.tar | docker import - test/ubuntu:v1.0`

> 之前的导入镜像和这个有点类似。实际上，既可以使用docker load 命令来导入镜像存储文件到本地的镜像库，也可以使用docker import 命令来导入一个容器快照到本地镜像库。两者的却别在于容器快照文件将丢弃所有的历史记录和元数据（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新制定标签等元数据信息。docker load是在之前save的时候镜像信息是什么，load之后也是同样的。
 

