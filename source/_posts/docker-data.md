---
title: Docker 数据管理
date: 2017-05-13 22:20:30
tags:
   - Docker
categories: Docker
---
> Docker 的数据管理应该核心中的核心。因为不管哪个公司，数据应该永远是第一位。

### 1,Docker 数据卷简介
Docker 的数据卷是一个可供容器使用的特殊目录，它绕过文件系统，提供很多的特性:
 * 数据卷可以在容器之间共享和重用
 * 对数据卷的修改会立马生效
 * 对数据卷的更新，不会影响镜像
 * 卷会一直存在，直到没有容器使用
数据卷的使用，类似于Linux下对目录或文件进行mount操作。
<!--more-->
### 2,容器内建数据卷
使用`docker run`命令的时候，使用`-v`标记可以在容器内创建一个数据卷，多次使用-v标记可以创建多个数据卷。
`docker run -d -P --name web -v /webdata centos /bin/bash`，这条命令成功的创建了一个名为web的容器(-P 是指允许外部访问容器需要暴露的端口)，但是我们用`docker exec -ti web bash` 或者`docker attach web`想进入到容器内部时，都会提示我们，容器未启动。之后运行`docker start web`,这在我们的理解中应该是启动了一个容器，但是但我们再想进入到容器时，发现还是提示我们容器未启动。用`docker ps`查看后台运行的docker容器，返现确实web没有运行。后来想了一下容器创建的步骤，容器确实创建了。但是我们想进入到容器里面却缺少交互界面。所以用了另一个命令`docker run -ti -d -P --name web -v /webdata centos /bin/bash`,加上`-ti`之后再用命令进入到容器就可以了。进入之后会发现在容器中根目录多了一个webdata目录。

#### 2.1,挂载宿主机的一个目录到容器
`-v` 标记也可以挂在一个本地的目录到容器里面作为数据卷。
`docker run -ti -d -P --name webdata -v /Users/alvin/test/docker/webdata:/share/webdata centos /bin/bash`，这条命令是指将本地`/Users/alvin/test/docker/webdata`目录挂载到webdata容器里面的`/share/webdata`目录。
如果要挂载多个目录，多次用`-v`标记就可以了。`docker run -ti -d -P --name webdata1 -v /Users/alvin/test/docker/webdata:/share/webdata -v /Users/alvin/test/docker/webdata:/share/webdata1 -v /Users/alvin/test/docker/webdata:/share/webdata2 centos /bin/bash` 将webdata,webdata1,webdata2都挂载到docker容器里面相应的位置。
如果想要对容器的数据卷目录进行权限控制的话，docker也是允许的，默认是rw权限。如果只想要可读权限。
`docker run -d -P -ti --name webdata -v /Users/alvin/test/docker/webdata:/share/webdata:ro centos bash`,目录到容器里面就只有读的权限了。
**这种方式特别适合挂载单个文件到容器里面**。
_`--volumes-from`参数所挂载的数据卷的容器自身并不需要保持在运行状态。_
如果要删除挂载了数据卷的容器，数据卷并不会被自动删除。如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时，显示的是使用`docker rm -v`命令来指定同时删除关联关联的容器。

#### 2.2,利用数据卷容器迁移数据
```
	docker run -ti --volumes-from dbdata --name db4 -v /Users/alvin/test/docker:/backup centos /bin/bash
	tar cvf /backup/backup.tar /dbdata
```
首先将dbdata容器内的数据卷和db4容器想关联，之后在宿主机和db4容器做数据卷共享，之后将数据卷容器的数据打包到和宿主机共享的位置。这样就做到了数据的备份了。

#### 2.3,恢复数据
 可以使用本地宿主机和docker文件夹共享。
 

