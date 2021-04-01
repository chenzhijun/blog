---
title: 过去一段时间遇到的一些问题
copyright: true
date: 2019-08-17 17:16:25
tags: 随笔
categories: 随笔
---

# 随笔-过去一段时间遇到的一些问题
<!--more-->
Q: Ansible 远程执行无法找到命令的问题。

A: 实际上是bash / sh的问题，执行的时候使用`ansible all -i hosts -m shell -a 'PATH="/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/bin:/usr/bin";source /etc/profile;groupadd docker'`

Q：kubernetes 一个节点的ip被加入到集群中，使得节点ip变成了kube-ipvs0里面的子网

A: 创建了外部服务，但是外部服务的ip地址为node的一个节点ip，产生的后果为这个节点与其它节点直接无法ping通。

Q: Docker 使用非Root用户管理或者 Docker无权限

A: https://docs.docker.com/install/linux/linux-postinstall/

`Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.26/build?buildargs=%7B%7D&buildbinds=null&cachefrom=%5B%5D&cgroupparent=&cpuperiod=0&cpuquota=0&cpusetcpus=&cpusetmems=&cpushares=0&dockerfile=Dockerfile&labels=%7B%7D&memory=0&memswap=0&networkmode=default&pull=1&rm=1&shmsize=0&t=harbor.uat.x.com%2Fx-library%2Fadms-app-0627%3A170&ulimits=null: dial unix /var/run/docker.sock: connect: permission denied`

Q: Docker 的远程仓库用户权限文件。

A: `$HOME/.dockercfg` 在用户根目录的`.dockercfg`目录。如果要查当前仓库登陆的用户`docker login hub.xxx.com`

Q：kubernetes 直接运行一个pod

A: `kubectl run testconfig --image=harob -o yaml --dry-run`

Q：kubernetes 获取某个资源的解释或者yaml的定义解释

A：`kubectl explain pods.spec`

Q: git 使用代理

A: 

```shell
git config --global http.proxy http://proxy.example.com:8888

git config --global --unset http.proxy

git config --global --add remote.origin.proxy "socks5://127.0.0.1:18001"

git config --unset-all
```

Q：docker删除不需要的镜像。

A: `docker images|grep 'none'|awk '{print $3}'|xargs -r docker rmi -f`

Q：MySQL 集群无法重启

A: 

```
show variables like 'wsrep_provider_options';
/data/mysql/grastate.dat 修改
safe_to_bootstrap: 1
```

Q: linux某个进程无响应

A: https://blog.csdn.net/jctian000/article/details/80695025

`/proc/进程号/fd`

`/proc/进程/stack`

Q：Linux逻辑卷扩容

A:

```shell

vgextend vg00 /dev/vdc
lvextend -L +20G /dev/vg00/home
xfs_growfs /dev/vg00/home

```

Q: curl timeout

A: `curl --connect-timeout 2 -m 5 100.66.7.2:25`