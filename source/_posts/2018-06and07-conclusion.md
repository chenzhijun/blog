---
title: 2018-06月至07月的一些总结
copyright: true
date: 2018-07-16 17:57:00
tags: 总结
categories: 总结
---

# 2018-06月至07月的一些总结

最近好久都没有写博客了。其实最近做的事情都比较单调，就是迁移环境，不过再迁移环境中，我还是收获了很多我缺少的。单从非技术方面，我缺少勉才的那股韧劲（问题不会过夜），缺少华哥那种细致（仔细的做过一遍，还会仔细的检查一遍，不是那种大致检查，是那种一条一条的比对），当然我也看到有些同事的缺点也是我的缺点，有则改之。

在搬迁中，华哥是我们部的总负责人，他对所有的事情都会有把控，搞不定的事情都会找他。而且他总能找到合适的方法解决。最重要的是，我犯错他却从来没有指责过，只是说怎样弥补，然后跟我们说正确的方法，而后我觉得挺对不住他的。总会想着把事情再做好点。勉才是小组的执行人，执行人真的超级强，问题基本没过夜，总是在12点的时候或者第二天一早问他，他就说问题解决了。。简直了。

这次搬迁我们是轮值的，也就是一开始将所有任务分配好，然后各自自己实验一次，写下步骤，下一轮再给另一个人按照写的步骤实行一次，这样来回3轮。最后基本上，步骤文档就没啥问题了。任何人都可以按照文档完整的搭出系统。
<!--more-->
这个过程会遇到一些问题，有些已经遗忘，有些所幸有所备份。

Q: Docker 新镜像里面脚本无权限
A: 这个问题是脚本在windows传递到linux系统里面可能会出现，如果看ls，那么X的权限都没有了。这个时候解决拌饭就是`chmod a+x xxx.sh`

Q: Docker 导入导出镜像
A：镜像保存：`docker save -o image-name.tar image-name:latest`，镜像导入：`docker load < image-name.tar`

Q: Docker 运行一个容器，执行完之后就退出
A: `docker run --rm image:latest bash/sh`

Q: 从运行中的容器拷贝文件/文件夹到宿主机
A: docker cp container-id:/path/to/file /path/to/file

Q: 修改Docker的docker.service
A：`sudo cat /lib/systemd/system/docker.service`,可以使用`find / -name docker.service`来找到相应文件。修改这个文件之后需要重启daemon。`systemctl daemon-reload`。在这个文件里面可以增加docker的http,https的代理。

Q: 遇到Docker push镜像的时候出现https的问题
A: 在/etc/docker/daemon.json文件中增加：["insecure-registries":["registry.chenzhijun.com:5000"]

Q: "can't create unix socket /var/run/docker.sock: is a directory"
A: rm -fr  /var/run/docker.sock/

Q: 容器内部无法访问外网：
A:  1：检查容器内部的dns，cat /etc/resolv.conf
    2：检查宿主机的dns，

Q: ubuntu18 重启无法进入桌面
A: 选择联网，让其然后下载显卡厂商的驱动。

Q: Linux统计一个文件夹下某类文件的数量
A: ls ./* | wc -l

Q: 查找当前目录使用情况：
A: du -h

Q: 查看磁盘使用情况：
A: df -h

Q: 修改文件所有者
A: chown -R user:group xxFile/xxDir

Q: shell 文件分割字符串
A:如下面的代码：

```shell
# !/bin/bash
name=$(hostname -i)
file=${name%% *} //分割空格
df -h >/home/rhlog/$file.log
```

Q: find 使用

Q: AWK 使用，grep 使用

Q: haproxy 使用

Q: Ansible 直接使用命令

A: ansible-playbook -i host  -m shell -a "docker ps"

Q: Dockerfile 制作镜像

