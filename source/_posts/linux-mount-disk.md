---
title: linux 挂载磁盘
copyright: true
date: 2018-07-26 13:18:45
tags: Linux
categories: Linux
---

# linux 挂载磁盘

最近需要挂载磁盘，记录一下。

一个500G的磁盘。使用`fdisk`查看，如果没有就使用`lsblk`可以查看到现在有哪个磁盘没有挂载。

之后就是挂载操作了。切换成root用户,查看磁盘使用的卷类型：ext4 , xfs
<!--more-->
![2018-07-26-13-23-23](/images/qiniu/2018-07-26-13-23-23.png)

1: 然后使用`mkfs -t xfs（或ext4） /dev/vdb`  

xfs,ext4 ：是指格式化成什么磁盘类型

/dev/vdb ：是指要挂载的磁盘

2: 挂载点 `mkdir /data` ，建立一个挂载点

3：挂载磁盘`mount /dev/vdb /data` 将刚刚格式化的磁盘挂载到`/data`目录

4：修改`/etc/fstab`文件，复制一行然后修改就可以了，将最后一个数字改为2。

![2018-07-26-13-33-04](/images/qiniu/2018-07-26-13-33-04.png)


## 挂卷快捷操作

用 lsblk 查看挂载的磁盘名：

```
sudo pvcreate /dev/vdb

sudo vgcreate vg02 /dev/vdb

sudo lvcreate -l 100%free -n data vg02

sudo mkfs.xfs -n ftype=1 /dev/vg02/data

sudo mkdir /data

sudo mount /dev/vg02/data /data

sudo echo /dev/mapper/vg02-data /data xfs defaults 0 0 >> /etc/fstab
```