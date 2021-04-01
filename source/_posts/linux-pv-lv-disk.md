---
title: Linux 创建和扩展逻辑卷
copyright: true
date: 2019-05-24 17:55:26
tags: 
 - pv
 - lv
 - disk
categories: Linux
---

# Linux 创建和扩展逻辑卷

最近遇到一个事，以前可能是给了一个大磁盘，然后我们全部格式化，一起挂载上去。后来发现用完了，扩展起来不是特别好扩展。
所以就找到了一个新的方式，我们使用逻辑卷来操作我们的磁盘。

## 准备

1. 一台centos7主机
2. 两块空盘

## 增加逻辑卷
<!--more-->
准备工组做好后，就可以慢慢操作了。

```shell
uname -a
Linux myhosts 3.10.0-957.5.1.el7.x86_64 #1 SMP Fri Feb 1 14:54:57 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```
### 查找空盘

使用`lsblk`或者`fdisk -l`

```shell

lsblk

NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda    252:0    0   60G  0 disk
└─vda1 252:1    0   60G  0 part /
vdb    252:16   0  500G  0 disk /data
vdc    252:32   0   10G  0 disk
vdd    252:48   0   10G  0 disk

-------

fdisk -l

Disk /dev/vda: 64.4 GB, 64424509440 bytes, 125829120 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00005822

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048   125821079    62909516   83  Linux

Disk /dev/vdb: 536.9 GB, 536870912000 bytes, 1048576000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/vdc: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/vdd: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


```

可以看到两块空盘:`/dev/vdc`,`/dev/vdd`。

### 创建pv

```shell
# pvcreate /dev/vdc
  Physical volume "/dev/vdc" successfully created
```

### 创建vg

```shell
# vgcreate vg00 /dev/vdc
  Volume group "vg00" successfully created
```

### 创建lv

```shell
[root@chenzhijun ~]# lvcreate -L 2g -n vg-data vg00
  Logical volume "vg-data" created.

[root@chenzhijun ~]#lvs -a
  LV      VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  vg-data vg00 -wi-a----- 2.00g
```

现在我们看一下我们创建pv,vg,lv

```shell
[root@chenzhijun ~]# pvs
  PV         VG   Fmt  Attr PSize  PFree
  /dev/vdc   vg00 lvm2 a--  10.00g 8.00g
[root@chenzhijun ~]# vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  vg00   1   1   0 wz--n- 10.00g 8.00g
[root@chenzhijun ~]# lvs
  LV      VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  vg-data vg00 -wi-a----- 2.00g
```
<!-->
lvcreate -l 100%free -n docker vg00
<-->
这个时候我们再进入到`/dev/mapper`目录，可以看到我们刚刚建立好的逻辑卷，卷组

```shell
[root@chenzhijun mapper]# pwd
/dev/mapper
[root@chenzhijun mapper]# ls
control  vg00-vg--data
```

### 格式化区为Linux可用的磁盘格式

```shell
[root@chenzhijun mapper]# mkfs.xfs /dev/mapper/vg00-vg--data
meta-data=/dev/mapper/vg00-vg--data isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

### 挂载到相应目录

Linux知道了这个可用空间的存在，现在我们要继续告诉它在哪里使用，也就是创建挂载点。
```shell
[root@chenzhijun /]# mkdir /mydata
[root@chenzhijun /]# ls -l
total 29288
drwxrwxrwx.   6 mwop mwop       77 Apr 10 19:23 app
lrwxrwxrwx.   1 root root        7 Dec 18  2015 bin -> usr/bin
dr-xr-xr-x.   4 root root     4096 Feb 25 15:21 boot
…………
drwxr-xr-x    2 root root        6 May 24 18:21 mydata
```

使用`fdisk -l`,这个时候可以看到有了一个新的空间,`/dev/mapper/vg00-vg--data`：

```shell
[root@chenzhijun /]# fdisk -l

Disk /dev/vda: 64.4 GB, 64424509440 bytes, 125829120 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00005822

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048   125821079    62909516   83  Linux

Disk /dev/vdb: 536.9 GB, 536870912000 bytes, 1048576000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/vdc: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/vdd: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/vg00-vg--data: 2147 MB, 2147483648 bytes, 4194304 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

下面我们要把这个磁盘挂载到我们刚刚创建的目录上，编辑`/etc/fstab`文件：`vi /etc/fstab`

```conf
#
# /etc/fstab
# Created by anaconda on Thu Dec 17 17:11:31 2015
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=fc1bfc5d-a5d1-4c3c-afda-167500654723 /                       xfs     defaults        0 0
/dev/vdb /data                       auto     defaults        0 2
/dev/mapper/vg00-vg--data /mydata                       auto     defaults        0 2
```

复制第一行`yyp`。然后照着修改，最后使用`mount -a`验证下。

```shell

[root@chenzhijun /]# vi /etc/fstab
[root@chenzhijun /]# mount -a
[root@chenzhijun /]#

```

现在再使用`df -h`就可以看到我们刚刚挂载的盘了。

```shell
[root@chenzhijun /]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
/dev/vda1                   60G   56G  4.9G  92% /
devtmpfs                   3.8G     0  3.8G   0% /dev
tmpfs                      3.9G     0  3.9G   0% /dev/shm
tmpfs                      3.9G  384M  3.5G  10% /run
tmpfs                      3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/vdb                   493G  116G  352G  25% /data
tmpfs                      783M     0  783M   0% /run/user/0
overlay                     60G   56G  4.9G  92% /var/lib/docker/overlay2/e919ab7a38f61e66a783503f1fe9ebfdecd79ce08f35b21547bedf6c803c5b4f/merged
shm                         64M     0   64M   0% /var/lib/docker/containers/71a340da3d3e6f4039623c0709026b3546a2a6b1635ff356fcea867c49159f6f/shm
overlay                     60G   56G  4.9G  92% /var/lib/docker/overlay2/c247fba1ff6be41ed3d3c030c9944006f750fb6e1ab162728169077b1093b894/merged
shm                         64M     0   64M   0% /var/lib/docker/containers/b163be285d664d334d81fa0411e41791001a47111d4124eec4c839cb3e907833/shm
overlay                     60G   56G  4.9G  92% /var/lib/docker/overlay2/2827d53a9dc37f826b16211a02795a5f44bd1d69d26d2b7874750004cf18e1ab/merged
shm                         64M     0   64M   0% /var/lib/docker/containers/8973a1b6f16ccd6adb03512acd3e786001b1e357256ecb51ddfcfae7c2b9624b/shm
overlay                     60G   56G  4.9G  92% /var/lib/docker/overlay2/e718bbb03b70866dd91b78a53c1c735edd8279ca3d11797ba4235d5b6e0ef659/merged
shm                         64M     0   64M   0% /var/lib/docker/containers/87cb442aa259b54b88cb11bbfa0f2c35bb328fbc84c3ec5772a92ee90980253a/shm
overlay                     60G   56G  4.9G  92% /var/lib/docker/overlay2/13733d71ac904b8f5dc6f06122627a9b54f3253c69b773c304968bdb200291bf/merged
shm                         64M     0   64M   0% /var/lib/docker/containers/ad876b38777e4a5dde9b1d7d0add1d7808a996b7c4015c55df4abd6078695041/shm
overlay                     60G   56G  4.9G  92% /var/lib/docker/overlay2/492dd394a21d652026f21a73029a1d83968e6e0e319460c80e032ee03d3ebd63/merged
shm                         64M     0   64M   0% /var/lib/docker/containers/24ced8c700f62c11ab8ff3b7559a8576f1a802693c3b532e4728e22256298257/shm
/dev/mapper/vg00-vg--data  2.0G   33M  2.0G   2% /mydata
```

一个挂载磁盘的操作就完成了。

## 扩容逻辑卷空间

刚刚我们是一个10G的盘只使用了2G，磁盘利用率肯定不足嘛，所以我们扩充到8G。命令如下：

先看到一个盘还剩下多少空间：

```shell
[root@chenzhijun ~]# vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  vg00   1   1   0 wz--n- 10.00g 8.00g
```

增加6G：

```shell
[root@chenzhijun ~]# lvextend -L +6G /dev/mapper/vg00-vg--data
  Size of logical volume vg00/vg-data changed from 2.00 GiB (512 extents) to 8.00 GiB (2048 extents).
  Logical volume vg-data successfully resized.
```

确认磁盘：

```shell
[root@chenzhijun ~]# xfs_growfs /dev/mapper/vg00-vg--data
meta-data=/dev/mapper/vg00-vg--data isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 524288 to 2097152
```

现在再看pv,vg,lv

```shell
[root@chenzhijun ~]# vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  vg00   1   1   0 wz--n- 10.00g 2.00g
[root@chenzhijun ~]# lvs
  LV      VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  vg-data vg00 -wi-ao---- 8.00g
[root@chenzhijun ~]# pvs
  PV         VG   Fmt  Attr PSize  PFree
  /dev/vdc   vg00 lvm2 a--  10.00g 2.00g
```

使用df看下磁盘空间：

```shell
[root@chenzhijun ~]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
/dev/vda1                   60G   56G  4.9G  92% /
devtmpfs                   3.8G     0  3.8G   0% /dev
tmpfs                      3.9G     0  3.9G   0% /dev/shm
tmpfs                      3.9G  400M  3.5G  11% /run
tmpfs                      3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/vdb                   493G  116G  352G  25% /data
tmpfs                      783M     0  783M   0% /run/user/0
overlay                     60G   56G  4.9G  92% /var/lib/docker/overlay2/e919ab7a38f61e66a783503f1fe9ebfdecd79ce08f35b21547bedf6c803c5b4f/merged
shm                         64M     0   64M   0% 
/dev/mapper/vg00-vg--data  8.0G   33M  8.0G   1% /mydata
```

## 扩容VG

如果现在加了一个新盘，比如刚刚说的`/dev/vdd`。查看一下：

```shell
[root@chenzhijun ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda             252:0    0   60G  0 disk
└─vda1          252:1    0   60G  0 part /
vdb             252:16   0  500G  0 disk /data
vdc             252:32   0   10G  0 disk
└─vg00-vg--data 253:0    0    8G  0 lvm  /mydata
vdd             252:48   0   10G  0 disk
```

然后增加vg：

```shell
[root@chenzhijun ~]# vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  vg00   1   1   0 wz--n- 10.00g 2.00g
[root@chenzhijun ~]# vgextend vg00 /dev/vdd
  Physical volume "/dev/vdd" successfully created
  Volume group "vg00" successfully extended
[root@chenzhijun ~]# vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  vg00   2   1   0 wz--n- 19.99g 11.99g
```

扩容之前的路径：

```shell
[root@chenzhijun ~]# lvextend -L +8G /dev/mapper/vg00-vg--data
  Size of logical volume vg00/vg-data changed from 8.00 GiB (2048 extents) to 16.00 GiB (4096 extents).
  Logical volume vg-data successfully resized.
[root@chenzhijun ~]# vgs
  VG   #PV #LV #SN Attr   VSize  VFree
  vg00   2   1   0 wz--n- 19.99g 3.99g
[root@chenzhijun ~]# xfs_growfs /dev/mapper/vg00-vg--data
meta-data=/dev/mapper/vg00-vg--data isize=512    agcount=16, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=2097152, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2097152 to 4194304

```

查看磁盘：

```shell
[root@chenzhijun ~]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
/dev/vda1                   60G   56G  4.9G  92% /
devtmpfs                   3.8G     0  3.8G   0% /dev
tmpfs                      3.9G     0  3.9G   0% /dev/shm
tmpfs                      3.9G  400M  3.5G  11% /run
tmpfs                      3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/vdb                   493G  117G  351G  25% /data
tmpfs                      783M     0  783M   0% /run/user/0
overlay                     60G   56G  4.9G  92% /var/lib/docker/overlay2/e919ab7a38f61e66a783503f1fe9ebfdecd79ce08f35b21547bedf6c803c5b4f/merged
shm                         64M     0   64M   0% /var/lib/docker/containers/24ced8c700f62c11ab8ff3b7559a8576f1a802693c3b532e4728e22256298257/shm
/dev/mapper/vg00-vg--data   16G   34M   16G   1% /mydata

[root@chenzhijun ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda             252:0    0   60G  0 disk
└─vda1          252:1    0   60G  0 part /
vdb             252:16   0  500G  0 disk /data
vdc             252:32   0   10G  0 disk
└─vg00-vg--data 253:0    0   16G  0 lvm  /mydata
vdd             252:48   0   10G  0 disk
└─vg00-vg--data 253:0    0   16G  0 lvm  /mydata

```

## lv删除

`lvremove `