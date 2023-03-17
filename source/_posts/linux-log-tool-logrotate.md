---
title: Linux 日志管理工具 - Logrotate
copyright: true
date: 2021-05-20 22:35:30
tags: Linux
categories: Linux
---

# Linux 日志管理工具 - Logrotate

最近在使用 openresty(nginx) 作为容器平台 kubernetes 的入口路由服务；流程大致为前端新建一个路由规则的时候，获取到 svc 的 endpoint 然后将其作为 nginx 的后端服务。实时监听 svc 的状态然后更新 ep 到 nginx；这里遇到一个问题，nginx 本身没有做到日志切割，在上线一段时间之后，nginx 的 vhost_access.log 日志文件竟然达到了 50G ，导致出现系统磁盘告警。google 一下没发现 nginx 有自带的日志轮转的功能（其实也有，只是要装插件和重新编译）；在线上最好的方式是什么了？最后采用了 Logrotate 来做日志切割。
<!--more-->
logrotate 的主要功能为定时对增长中的日志文件进行自动切割，压缩，删除等操作；可以将定时任务设置为日，周，月，也可以自定义 crontab 的时间执行。

## Logrotate 安装与配置文件

首先需要安装 Logrotate： `yum install -y logrotate`; 安装完之后可以看到，默认是在`/etc/cron.daily/logrotate` 有一个定时执行任务。`/etc/cron.daily/logrotate`:

```shell
#!/bin/sh

/usr/sbin/logrotate -s /var/lib/logrotate/logrotate.status /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
```

另外我们可以看到它的配置文件:

![2020-05-20-22-59-26](/images/qiniu/2020-05-20-22-59-26.png)

它的配置主要在`/etc/logrotate.conf`文件以及`/etc/logrotate.d/`目录下。

```conf
# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
	minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

```

打开它的主配置文件`/etc/logrotate.conf`可以看到它的一个配置：`include /etc/logrotate.d`，所以我们的配置基本上可以放到这个`logrotate.d`目录下。

## Logrotate 实际配置与操作

在`/etc/logrotate.d`目录下我们可以新建一个`ingress`文件，内容如下：
```conf
# /data/caas/ingress-gateway 目录下的 *log 文件都会进行正则匹配
/data/caas/ingress-gateway/*log {
    # 先进行 copy,然后进行 truncate，这里注意，在 truncate 的时间可能有一部分无法 copy 所以导致丢失数据。
    copytruncate
    # 服务/etc/
    daily
    # 创建新文件
    create
    # 保留切割的历史文件
    rotate 100
    # 正则匹配下一个 log 切割出错，继续执行，不然会中断
    missingok
    # 非空才执行
    notifempty
    # 对切割的日志进行压缩
    compress
    # 对于满足条件的 log，只运行一次脚本
    sharedscripts
    # 日志大小满足 400M 进行切割
    size 400M
    # 创建新目录用来存储旧的日志
    createolddir
    # 旧的日志目录
    olddir /data/caas/ingress-gateway/bak
    # 切割后的文件的带时间后缀，不然默认是log.1，log.2
    dateext
    # 使用 dateext 时候的日志格式
    dateformat -%Y%m%d%H
}
```

目录下的配置项如果没有的会使用 `/etc/logrotate.conf` 的默认配置，如果有的会覆盖掉全局配置使用自己定义的配置。在我上面定义的是一个将文件进行备份后进行 truncate 的操作，这种操作的好处就是容器与日志文件的句柄不会断。当然还有另一种方式：

```conf
/data/caas/ingress-gateway/*log {
    #copytruncate
    daily
    create
    rotate 100
    missingok
    notifempty
    compress
    sharedscripts
    size 400M
    createolddir
    olddir /data/caas/ingress-gateway/bak
    dateext
    dateformat -%Y%m%d%H
    # 日志切割完之后进行执行脚本，必须空行
    postrotate
       docker ps|grep ingress-gateway|grep -v pause|awk '{print $1}'|xargs docker rm -f
    endscript
    # 和 postrotate 进行对应
}
```

强制执行方式：

`logrotate -fv /etc/logrotate.d/ingress3`：

```shell

-f 强制执行
-d debug 模式
-v 显示输出
-s 状态文件路径

```



<!--
#config logrotate
cat > /etc/logrotate.d/ingress << EOF
/data/wisecloud/ingress-gateway/*log {
    copytruncate
    daily
    create
    rotate 15
    missingok
    notifempty
    compress
    sharedscripts
    size 1G
    createolddir
    olddir /data/wisecloud/ingress-gateway/bak
    dateext
    dateformat -%Y%m%d%H
}
EOF
-->