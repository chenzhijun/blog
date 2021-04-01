---
title: Etcd 集群搭建
copyright: true
date: 2019-05-23 22:02:39
tags: Etcd
categories: Etcd
---

# Etcd 集群安装

本文参考：https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/clustering.md

不得不说etcd的安装真的非常容易。在https://github.com/etcd-io/etcd的release里面找到相应的版本。然后直接二进制启动，一个单节点就好。真的是太简单啊。 

不过搭建集群版本的话还是需要做一些配置，下面就是我用集群搭建的环境并且真实可用的过程。

下载好相应版本的etcd，然后做如下配置。我们采用的方式是使用Linux的systemd服务：
<!--more-->
```conf
[Unit]
Description=etcd service
Documentation=https://github.com/coreos/etcd

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd --name infra0 --initial-advertise-peer-urls http://100.69.216.107:2380 \
--listen-peer-urls http://100.69.216.107:2380 \
--listen-client-urls http://100.69.216.107:2379,http://127.0.0.1:2379 \
--advertise-client-urls http://100.69.216.107:2379 \
--initial-cluster-token etcd-cluster-1 \
--initial-cluster infra0=http://100.69.216.107:2380,infra1=http://100.69.216.108:2380,infra2=http://100.69.216.109:2380 \
--initial-cluster-state new \
--data-dir /data/etcd \
--heartbeat-interval 1000 \
--election-timeout 5000
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

然后我们使用`systemctl start etcd.service`一个节点就起来了。另外的2个节点也是一样的操作方式，不过要注意：

 1. --name 这里要注意不一样，每个集群的名字都是独立的
 2. --initial-cluster-token 这个是每个集群同一个名字
 3. --initial-cluster-state 这个如果是新集群就是new
 4. --data-dir /data/etcd 这个是一定要有，一定要先创建。不然etcd会在启动命令的目录自己建立一个name.etcd的数据目录，而且如果下次修改了目录，这个节点加入到集群还有坑


ps: 遇到的一些问题：

一定要指定data-dir，防止手动测试的时候加入了集群，那么下次指定data-dir之后，该节点就无法加入集群：member 9b3523b532ddb797 has already been bootstrapped 这就是因为之前已经加入了集群，然后data目录下跟当前设置data-dir不一样。解决方式就是将之前的name.etcd目录下的member文件放到新的data-dir下面
