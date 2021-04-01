---
title: Linux cron clean log files older than N days
copyright: true
date: 2018-10-29 22:14:21
tags: Linux
categories: Linux
---

# 使用linux crontab定时清理n天前的日志文件

最近有个需求，需要在linux机器上定时执行清理n天前的日志文件。其实我开始做了个更有意思的清理工具，根据alertmanager做webhook，然后在每个Linux机器上开启一个agent，收到请求再执行清理。不过使用crontab也是一个非常有用的工具。

crontab是Linux的一个守护进程，定时执行的工具。详细的内容可以使用`man crontab`查看。废话不多说，直接来看怎么使用它。只有用起来，才是属于自己的。
<!--more-->
## 查看当前有哪些定时任务

`crontab -l` 查看当前已经存在的定时任务。

![2018-10-29-22-26-45](/images/qiniu/2018-10-29-22-26-45.png)

可以看到，你需要的就是准备一个shell脚本（任务指令），一个定时时间（执行频率），一条触发指令（程序入口）。
准备好这三个东西就可以。

下面我们以一个简单的需求来演示。

需要删除 */data* 目录下文件名存在的带 *log* 的文件，修改这些文件的大小为0。

## 设置一个新的定时任务

`crontab -e` 可以进入crontab编辑页面。将带‘*’的那行复制成新的行，如下：

![2018-10-29-22-33-33](/images/qiniu/2018-10-29-22-33-33.png)

可能会问，前面的‘*’的时间怎么设置。可以使用`cat /etc/crontab`查看解释：

![2018-10-29-22-35-12](/images/qiniu/2018-10-29-22-35-12.png)

之前的需求，我将`clean3dayslog.sh`文件放到`/root`目录下，然后执行文件，clear3dayslogs.sh的内容为：

```shell

#!/bin/bash
find /data -mtime +3 -name "*.log*"|grep rtlog|xargs -i truncate -s 0 {}
```

这个demo比较简单，不过用处确实很大，一般清除的都在开发环境上，日志输出。