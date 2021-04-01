---
title: 最近遇到的小问题总结-2018522
copyright: true
date: 2018-05-22 18:47:28
tags: problem
categories: problem
---

# 最近遇到的小问题总结-2018522

1. Excel将两列合并成一列：`=CONCATENATE()`

2. linux统计文件,[原文链接](http://blog.sina.com.cn/s/blog_464f6dba01012vwv.html):

```shell
统计某文件夹下文件的个数
ls -l |grep "^-"|wc -l

统计某文件夹下目录的个数
ls -l |grep "^ｄ"|wc -l

统计文件夹下文件的个数，包括子文件夹里的
ls -lR|grep "^-"|wc -l

如统计/home/han目录(包含子目录)下的所有js文件则：
ls -lR /home/han|grep js|wc -l 或 ls -l "/home/han"|grep "js"|wc -l

统计文件夹下目录的个数，包括子文件夹里的
ls -lR|grep "^d"|wc -l
```

3. linux sort的一些用法：

```shell
去除重复行
sort file |uniq

查找非重复行
sort file |uniq -u

查找重复行
sort file |uniq -d

统计
sort file | uniq -c
```

3. 什么是linux vip：[https://www.zhihu.com/question/67682565](https://www.zhihu.com/question/67682565)，[https://www.novell.com/documentation/bcc/bcc11_admin_nw/data/bq7ucwl.html](https://www.novell.com/documentation/bcc/bcc11_admin_nw/data/bq7ucwl.html)。

4. Linux scp命令，上传或下载文件

复制本地文件到远程服务器：`scp local_file remote_username@remote_ip:remote_folder`；
例如：`scp /home/chen/file.md chen@192.168.1.1:/home/chen/test.md`

复制远程文件到本地：`scp remote_username@remote_ip:remote_folder local_file`
例如：`scp chen@192.168.1.1:/home/chen/test.md /home/chen/file.md`

5. 修改文件所属用户和组信息

`chown username:usergroup /path/to/file`

6. 修改文件权限: ugo,a
user,group,other,all

`chmod a+x /path/to/file`

`chmod u+w /path/to/file`

7. linux 查看ip：

```shell
ip addr
ifconfig
hostname -I
```

7. 解压缩文件：

```
解压文件：tar -zvxf xxx.tar.gz

压缩文件： tar -zvcf xxx.tar.gz /path/to/file/tozip
```

8. vim 一些操作

```shell

vim 全文件内替换  ： :%s/old/new/g
当前行替换：:s/old/new/g


vim 打开多个文件： split/vsplit     
文件切换ctrl w w， 获取CTRL w (上下左右或者jkhl)，CTRL F6 6

```

9. docker 删除容器状态为exited的容器：

`docker rm -v $(docker ps -aq -f status=exited)`

10. 删除所有none的tag的镜像

`docker images |grep none |awk '{print $3}'|xargs -i docker rmi {}`

11. idea重置配置文件。

D:\Users\chenzj001\.IntelliJIdea2018.1 删除这个文件就好了，全部重置。也可以只删除config。
windows找到用户-->.i

也就是在`用户目录`下,删除`.IntelliJIdea2018`

12. windows 查找端口占用进程

```
netstat -aon|findstr "端口号"

tasklist|findstr “端口号”
```
