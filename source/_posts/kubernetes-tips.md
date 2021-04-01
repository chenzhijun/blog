---
title: ops 操作手册
copyright: true
date: 2019-12-01 18:09:17
tags:
categories:
---


## 给命令行增加快捷操作提示符

```shell
source <(kubectl completion bash) # setup autocomplete in bash into the current shell, bash-completion package should be installed first.echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
alias k=kubectl
complete -F __start_kubectl k
```
<!--more-->
## 快速删除kubernetes资源

删除Terminating的pods

`kubectl get pods --all-namespaces|grep Termi|awk '{print "kubectl -n "$1" delete pods "$2" --force --grace-period=0"}'|xargs -i echo {} >delete.sh`

缩容：

`kcc scale --current-replicas=3 --replicas=0 deployment/orche`

禁止主机调度：

`kubectl uncordon NODE_NAME`

显示节点ip：

`kubectl get nodes -o wide --show-labels|awk '{print $1"\t"$2"\t"$6}'`

删除标签：

 `kubectl label node cnsz12.company.cn bad-`

快速启动一个容器：

 `kc run -i --tty --image harbor.com/library/busybox:1.28.4 dns-test --restart=Never --rm /bin/sh`

## 抓包工具

```shell

https://github.com/buger/goreplay

./gor --input-raw :8080 --input-raw-track-response --output-stdout

./gor --input-raw :8080 --output-stdout --http-allow-url RH_SSO/SeqSso.sso

tcpdump -i eth0 dst host 10.70.1.76 and dst port 4410

```

## 文件底层无法编辑

chattr


## docker 容器网络工具镜像

`docker run -ti --net container:a967 nicolaka/netshoot:latest bash`


<!--
jdk8u202之后，jvm获取cgroup的内存限制。
kubectl get pods --all-namespaces|grep Termi|awk '{print "kubectl -n "$1" delete pods "$2" --force --grace-period=0"}'|xargs -i echo {} >delete.sh





网络模型：ovs open vswitch ,daemonset启动
openshift本身使用static pod
find /sys/fs/cgroup/memory -type d | wc -l

The node was low on resource: memory. Container rdspm-app was using 2159200Ki, which exceeds its request of 0


https://www.cnblogs.com/duanxz/p/10247494.html

https://blog.csdn.net/weixin_33744141/article/details/86251459


find . -amin -10 # 查找在系统中最后10分钟访问的文件
find . -atime -2 # 查找在系统中最后48小时访问的文件
find . -empty # 查找在系统中为空的文件或者文件夹
find . -group cat # 查找在系统中属于 groupcat的文件
find . -mmin -5 # 查找在系统中最后5分钟里修改过的文件
find . -mtime -1 #查找在1天以内修改过的文件
find . -mtime +7 #查找在7天以外修改过的文件
find . -nouser #查找在系统中属于作废用户的文件
find . -user fred #查找在系统中属于FRED这个用户的文件
find . -type -f # 查找文件类型为普通文件的文件


nginx 会话粘贴


http://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive
https://stackoverflow.com/questions/24453388/nginx-reverse-proxy-causing-504-gateway-timeout
https://superuser.com/questions/1489355/website-shows-a-blank-page-when-opened-from-search-engines-or-href-links-but-wo


应用频繁启动，导致k8s node节点not ready

100.70.88.44


PLEG unhealth   https://github.com/kubernetes/kubernetes/issues/45419
https://github.com/kubernetes/kubernetes/issues/61117


CREATE USER 'monitor'@'100.69.224.39' IDENTIFIED BY 'monitor1875' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'monitor'@'100.69.224.39';
GRANT SELECT ON performance_schema.* TO 'monitor'@'100.69.224.39';

go mod

haproxy 支持websocket

sed -i '/120/d’ known_hosts 替换120的行

sed -n ‘/120/p' known_hosts  查找ruby的行

git log --graph --pretty=oneline --abbrev-commit

Control a 最开始，control e 最末尾

nodeport,hostport
-->