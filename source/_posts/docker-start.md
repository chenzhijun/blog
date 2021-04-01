---
title: Docker简单入门-搭建 JavaWeb Docker 运行环境
date: 2017-04-12 14:07:29
tags:
	- Docker
categories: Docker
---
### Docker 入门
> 容器技术已经越来越火爆，作为攻城狮有必要了解一下Docker

### 什么是Docker

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。正如 Docker 的 Logo 所示。一条大鲸鱼载着各种集装箱。大鲸鱼就可以看成宿主机，集装箱就是在宿主机上的容器，容器之间是相互隔离的。
<!--more-->

### 怎么用Docker

Docker 最重要的的是 image ，也就是镜像。先有镜像，再建立容器。

Docker 的使用方式是很简单的，暂时我也只是刚刚学会一些简单的使用方式。暂时先介绍 Docker 几个命令的使用方式：

* docker --version    查看当前安装的 Docker 的版本
* docker --info       查看 Docker 的信息
* docker images       查看所有的镜像
* docker ps -a        查看所有的容器
* docker ps           查看正在运行的容器
* docker rm <container name>  删除容器
* docker rmi  <image name>    删除镜像    多加一个i，rmi。
* docker stop <container name>    停止运行的容器
* docker start <container name>   启动一个容器
* docker run -ti [--name container-name] -v [宿主机地址]:[容器地址] <镜像名> /bin/bash   eg:docker run -ti --name web -v /Users/alvin/Devtools/docker/:/mnt/software/ centos /bin/bash    这条命令的意思在 centos 镜像上建立一个名为 web 容器。-v 的意思是在宿主机的/Users/alvin/Devtools/docker/ 挂载在 web 容器的/mnt/software/ 位置。
* docker commit <容器ID> chenzhijun/javaweb:1.0  保存容器为一个镜像


### 怎么创建镜像

一切从理论都是屁话，理论结合实践才是王道。

下面建立一个Javaweb运行环境的容器，生成image上传到hub.docker.com。

1:首先要下载Centos 镜像
	```docker pull centos```

2:在Tomcat官网上下载Tomcat linux 版本；在 Oracle 官网下载 JDK 。解压后将tomcat和jdk存放在本地宿主机 [自定义目录:/User/alvin/Devtools/docker/] 


3:在centos上建立一个新容器
	```docker run -ti --name chenzhijun -v /Users/alvin/Devtools/docker/:/mnt/software/ centos /bin/bash```
	
4:进入容器里面，将software下的tomcat和jdk移动到/opt/下面

```
Vim ~/.bashrc 
# 在.bashrc 最后面加入下面两行
export JAVA_HOME=/opt/jdk
export PATH=$PATH:$JAVA_HOME
```

5:进入容器后创建脚本

```
vi /root/run.sh

#!/bin/bash
source ~/.bashrc
sh /opt/tomcat/bin/catalina.sh run

```

6:修改脚本文件权限

```
chmod u+x /root/run.sh
```

7:保存容器到镜像

```
1:查看容器的id
docker ps -a

2:保存容器为镜像
docker commit (容器id) chenzhijun/javaweb:0.1

3:启动镜像
docker run -d -p 58080:8080 --name javaweb chenzhijun/javaweb:0.1 /root/run.sh
启动一个容器，并且容器运行的时候运行脚本，绑定宿主机的端口58080到容器的端口8080，运行的容器名字为javaweb
```

8:上传镜像到hub.docker.com,首先保证在hub上有一个相同名字的镜像地址，跟git类似。

```
docker login
username:
password:

docker push chenzhijun/javaweb
```
好了，暂时照着这个操作就基本上可以了。 如果遇到问题可以给我留言，也可以联系我的邮箱：vbookchen@gmail.com





