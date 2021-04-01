---
title: 重启Linux主机后自动运行任务或者脚本
copyright: true
date: 2018-10-30 21:13:40
tags: Linux
categories: Linux
---

# 重启Linux主机后自动运行任务或者脚本

有时候我们在主机上做了一些agent应用，这些应用平常都是主机启动，agent就需要启动。相当于“伴生”。尽管第一次或者第二次我们能依靠记忆或者自我约束来启动这些agent，但是有时候还是会免不了忘记。那么有没有办法让这种agent做成开机启动呢？下面提供两种Linux设置开机启动应用的方法。

<!--more-->
## 1. crontab 实现脚本(应用)开机启动

crontab的介绍在我的另一个博客里面[crontab使用linux crontab定时清理n天前的日志文件](http://chenzhijun.me/2018/10/29/linux-crontab-clean-ndays-log/)，这里我们介绍一个使用它来实现开机启动的用法。

使用`crontab -e`，在crontab的编辑页面增加下面的内容:`@reboot sleep 10 && bash /root/test.sh`，sleep 10 是指在开机10秒后启动，启动的脚本是`/root/test.sh`。里面的内容，可以自己根据需要编写。最后要记住 **将test.sh脚本设置为可执行权限 744 或者 777 或者 a+x**。这样一个简单的开启启动任务就完成了。

## 2. systemd 服务实现脚本(应用)开机启动

systemd 是linux机器的一个daemon服务。主要是创建一个`.service`文件，然后进行开机启动。下面来看一下实际使用过程。

`vi /etc/systemd/system/auto_exe.service`

```conf

[Unit]
Description=auto exec after reboot

[Service]
Type=oneshot
ExecStart=/usr/local/bin/auto.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

```

一个systemd的`service`由三部分构成:Unit,Service,Install。详细的内容或者介绍可以使用man，或者google。

我这里表示的是在机器启动后，执行`/usr/local/bin/auto.sh`。

auto.sh，记得将权限赋值为`a+x`，777，或者744
```shell

#!/bin/bash
echo "hello" >> /home/auto_exe.log
```

之后使用`systemctl daemon-reload`刷新service，我们可以使用`systemctl start auto_exe` 进行调试，看脚本是否执行。
使用`systemctl status auto_exe`查看服务状态。如果成功，那么就可以加入确保启动执行脚本，使用`systemctl enable auto_exe`，这样就设置好了。

简单的linux应用开机启动就设置好了。