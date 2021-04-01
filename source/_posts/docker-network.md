---
title: Docker 网络
date: 2017-05-14 18:33:32
tags:
   - Docker
categories: Docker
---

> 有了底层，有了数据，还差一个网络基础配置，那样就完美了

### 1,宿主机和Docker容器端口对应
`docker run -ti --name test-network -d -p 50001:8080 chenzhijun/javaweb:1.0` 或者用 `docker run -ti --name test-network -d -p 50001:8080 chenzhijun/javaweb:1.0 /root/run.sh`

创建一个在chenzhijun/javaweb:1.0镜像上的test-network容器，并且将本地的50001端口映射到docker的8080端口。chenzhijun/javaweb:1.0 镜像可以在hub.docker.com上下载，进入到容器后，进入root文件夹下，启动run.sh。然后再本地就可以用localhost:50001端口访问了。
<!--more-->
### 2,映射所有接口地址

`docker run -ti --name network -d -p 127.0.0.1::8080 chenzhijun/javaweb:1.0`
这样就是在本地随机分配了一个端口映射到docker的8080。
```
	> docker port network
	> 8080/tcp -> 127.0.0.1:32768
	> docker inspect network
```

### 3,容器互联实现容器间通信
容器的连接系统是除了端口映射外另一种可以与容器中应用进行交互的方式。它会在源和接受容器之间创建一个隧道，接收容器可以看到源容器制定的信息。
`docker run -d -P --name web --link name:alias images-name`,其中name是要链接的容器的名称，alias是这个链接的别名。
`docker run -it -d --name web2 --link db:mydb centos bash`创建一个和db容器有关联,在web2容器/etc/hosts中文件内容为:
```
[root@a1307bd12135 /]# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.6	mydb d121bb075116 db
172.17.0.5	a1307bd12135
```

> 使用Docker快速掌握新技术要点并完成适当的技术储备