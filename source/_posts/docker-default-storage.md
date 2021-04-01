---
title: Docker修改默认存储空间
copyright: true
date: 2019-05-24 17:20:52
tags: Docker
categories: Docker
---

# Docker 修改默认存储空间

最近碰到一个问题，由于需要下载很多的镜像文件，如果使用docker安装时候的默认存储空间，那肯定是不行的。默认的存储空间可以使用：`docker info`看到：在输出的信息里面会发现:`Docker Root Dir: /var/lib/docker`；这个目录其实就是在根目录下吗，如果下载的镜像数量过大并且又大，那么肯定是的完蛋的~~~

所以就需要修改docker的存储空间。
<!--more-->
我们的版本是：

```shell
docker -v
Docker version 1.13.1, build 07f3374/1.13.1
```

所以对于1.13+以上的版本应该都是没问题的，1.13以下的你可以自己尝试下，不一定能行，也不一定说不能行。

在前面：<font color="red">修改有风险，操作需谨慎</font>，在我修改了目录之后，容器和镜像文件都不能使用`docker ps/image`看到了，所以操作前一点要备份好。当然你把存储路径改回去当然也是可以看到的。

## 方式一：修改/etc/docker/daemon.json

修改这个json文件是最简单也是最方便的，没有之一。修改如下：

```json
{
    "insecure-registries": [
        "registry.xxx.com"
    ],
    "graph": "/docker-data/docker"
}
```

最重要的就是`"graph": "/docker-data/docker"`，这个就是指定存储位置。

## 方式二：修改/usr/lib/systemd/system/docker.service

修改docker的启动命令也能做到将存储默认位置修改。

```conf
[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.com
After=network.target
Wants=docker-storage-setup.service
Requires=docker-cleanup.timer

[Service]
Type=notify
NotifyAccess=main
EnvironmentFile=-/run/containers/registries.conf
EnvironmentFile=-/etc/sysconfig/docker
EnvironmentFile=-/etc/sysconfig/docker-storage
EnvironmentFile=-/etc/sysconfig/docker-network
Environment=GOTRACEBACK=crash
Environment=DOCKER_HTTP_HOST_COMPAT=1
Environment=PATH=/usr/libexec/docker:/usr/bin:/usr/sbin
ExecStart=/usr/bin/dockerd-current \
          --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current \
          --default-runtime=docker-runc \
          --exec-opt native.cgroupdriver=systemd \
          --userland-proxy-path=/usr/libexec/docker/docker-proxy-current \
          --init-path=/usr/libexec/docker/docker-init-current \
          --seccomp-profile=/etc/docker/seccomp.json \
          $OPTIONS \
          $DOCKER_STORAGE_OPTIONS \
          $DOCKER_NETWORK_OPTIONS \
          $ADD_REGISTRY \
          $BLOCK_REGISTRY \
          $INSECURE_REGISTRY \
          $REGISTRIES
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity
TimeoutStartSec=0
Restart=on-abnormal
KillMode=process

[Install]
WantedBy=multi-user.target

```
你可以看到`DOCKER_STORAGE_OPTIONS`,然后查看文件:`/etc/sysconfig/docker-storage`。在`DOCKER_STORAGE_OPTIONS="--storage-driver overlay2 "`上加上`DOCKER_STORAGE_OPTIONS="--storage-driver overlay2 --graph /docker-data/docker` 然后使用`systemctl daemon-reload`,`systemctl restart docker`就可以了。

当然如果你直接在`docker.service`文件里面改也是可以的，放到配置文件里面也行，都ok。想怎样就怎样~~