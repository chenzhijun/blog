---
title: Docker 镜像 
date: 2017-05-13 12:52:33
tags:
   - Docker
categories: Docker
---

## 镜像是Docker的三大核心概念之一。

> Docker运行容器前需要本地存在对应的镜像，Docker会尝试先从默认镜像仓库下载，可以自定义默认仓库位置。

### 1，获取镜像
`docker pull name[:TAG]`,如果TAG没有的话，默认是latest。
获取到镜像后，就可以使用镜像创建容器，在其中运行bash应用。

### 2，查看镜像信息
`docker images`,列出本地主机上已有的镜像。
`docker tag dl.dockerpull.com:5000/ubuntu:latest ubuntu:latest`,给本地镜像添加新标签,新标签的镜像id是一样的。 
`docker inspect image-id`,获取image-id的详细信息，json串。如果只想要某个信息的话，可以用
`docker inspect -f {{".RootFS"}} 48b5`,`RootFS`是json的某一项内容的详情。`48b5`,是镜像id。
<!--more-->
### 3，搜寻镜像
`docker search image-name`,搜索远端仓库中共享的镜像。
* --automated=false 仅显示自动创建的镜像
* --no-trunc=false 输出信息不截断显示
* -s, --start=0 指定仅显示评价为指定星级以上的镜像

### 4，删除镜像
使用镜像的标签删除镜像
`docker rmi image [image...]`,其中的image可以为标签或ID;
`docker rmi image:tag`,删除某个标签为tag的image;不带tag时默认为latest;
如果有容器是以要删除的镜像为基础创建的，那么镜像文件默认是无法被删除的。
如果要强行删除一个容器，可以用`docker rmi -f image:tag`,最好是先将基于镜像的容器删除掉`docker rm container-id`，之后再去删除镜像。

### 5，创建镜像
创建镜像的方法有三种: 基于已有镜像的容器创建，基于本地模板导入，基于Dockerfile创建。

a)基于已有容器创建
`docker commit [OPTIONS] container [REPOSITORY[:TAG]]`:
* -a, --author="" 作者信息。
* -m, --message="" 提交信息。
* -p, --pause=true 提交时暂停容器运行。

eg:`docker commit -m "added test images" -a "chenzhijun" 7e036 newimage:1.0` 以容器id为7e036*的容器为基础创建一个image。image-name为newimage,image-tag为1.0，如果1.0为空，默认为latest。

b)基于本地模板导入
本地先下载了一个`apache-tomcat-7.0.75.tar.gz`,然后使用`cat apache-tomcat-7.0.75.tar.gz |docker import - my-tomcat:1.0`这样就可以创建一个本地`my-tomcat`镜像。

### 6，存出和载入镜像
使用docker save 和 docker load命令来存出和载入镜像
a)存出镜像
`docker save -o my-name.tar image[:tag]`
这样就能将image存出到一个my-name.tar的文件下了

b)载入镜像
`docker load laod --input my-name.tar`
`dcoker load < my-name.tar`

### 7，上传镜像
`docker push NAME[:TAG]`
```
	docker tag test:latest user/test:latest
	docker push user/test:latest

	-- 可能需要登录
```




