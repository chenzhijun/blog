---
title: Docker 快速搭建 MySQL 和 Redis
copyright: true
date: 2018-12-16 20:04:12
tags: Docker
categories: Docker
---

# Docker 快速搭建 MySQL 和 Redis

## MySQL 环境搭建

最近做开发的时候遇到一个比较有意思的事情，如何搭建一个数据库，还有相关的redis等。因为没有外网，不可能说直接yum安装，又不想到每个官网去找相应的安装包。自然的，就想到了用Docker。一开始用到docker安装一个mysql的时候确实也是非常方便，比如一个`docker run`命令就启动了一个mysql，但是开发到一部分的时候，发现。我去！怎么又乱码？？ 咦，怎么市区也不对了？GG，发现还有很多小问题。今天有空一并总结下，下次如果有这种事就可以直接用了。

### 使用Docker启动启动Mysql容器

如果需要一个mysql数据库，直接使用docker来运行一个容器：`docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root123456 mysql:5.7`

这样你就可以在本地ip+3306端口来访问一个mysql数据库了，root密码是：`root123456`。第一次使用docker的时候就是因为这个原因被吸引了。想想如果本地安装需要做多少配置，而是用docker一条命令就帮你把那些复杂的操作都隐藏了。这种便利性，我想谁都不会说不想要。

不过虽然便利是便利了，但是还是需要注意一些问题：

<!--more-->

#### 字符问题

mysql默认其实是latin的字符集，docker 启动mysql的时候其实也是使用的默认字符。而我们做开发一般都是使用UTF-8的字符集，那出现这种情况该如何更改配置了？可以在启动的时候加上两个变量：`docker run -d -e MYSQL_ROOT_PASSWORD=root123456 mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci`

#### 数据磁盘问题

一个容器，如果那天不小心删除了，然后你重建就会发现之前的数据没有了，这种情况当然是不行啦。那么怎么将数据盘挂载出来：加上 `-v`，让存储在容器里面的数据存储到本地自定义的盘中：`-v /data/mysql:/var/lib/mysql`。

#### 默认数据库，以及时区修改

有的时候我们会需要导入一些表或者一些数据，这个时候该怎么操作? 其实也有一个环境变量：`MYSQL_DATABASE`;然后将数据库的初始化脚本放到`/docker-entrypoint-initdb.d/`目录下也就是将sql挂载到该目录下：

```shell
docker run -d --name mysql -p 13306:3306 -e MYSQL_ROOT_PASSWORD=root123456 -e MYSQL_DATABASE=DB_USER -e TZ=Asia/Shanghai -v $PWD/sql-scripts/:/docker-entrypoint-initdb.d/ -v /data/mysql:/var/lib/mysql mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

将数据库的sql放到当前目录的`sql-scripts/`目录下。

<!--- docker run -d --name test -p 13306:3306 -e MYSQL_ROOT_PASSWORD=root123456 -e MYSQL_DATABASE=DB_USER -e TZ=Asia/Shanghai -v $PWD/sql-scripts/:/docker-entrypoint-initdb.d/ -v /data/mysql:/var/lib/mysql -v /etc/localtime:/etc/localtime:ro mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

docker run -d --name test -p 13306:3306 -e MYSQL_ROOT_PASSWORD=root123456 -e MYSQL_DATABASE=DB_USER -e TZ=Asia/Shanghai -v /etc/localtime:/etc/localtime:ro -v $PWD/sql-scripts/:/docker-entrypoint-initdb.d/ -v /data/mysql:/var/lib/mysql -v /etc/localtime:/etc/localtime:ro mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
--->


## Redis 环境搭建

redis的搭建其实要比mysql要简单些，毕竟redis我们一般都只是用来当作缓存，而不会将数据持久化，所以只需要将一个容器run起来就可以了。不过我们一般会有redis的密码需要，所以完整的命令如下：

`docker run -d --name redis -p 6379:6379 redis:latest --requirepass "123456"`

这样redis就启动了，如果需要开机启动，加上`--restart=always`。

好了，今天一篇搭建mysql和redis的过程就到这里了。