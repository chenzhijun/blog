---
title: Windows使用sshkey连接GitHub
copyright: true
date: 2018-07-17 09:27:38
tags: github
categories: ssh
---

# Windows使用sshkey连接GitHub

经常连接github上传东西，经常push的时候需要输入用户名和密码。下面是使用ssh-key免密的方式访问自己的github仓库。

首先在本地生成ssh-key:

进入到`~/`,用户根目录，查看`.ssh`文件夹是否有相关的公钥私钥
如果没有新生成：

```
ssh-keygen -t rsa -C "xxx@yy.com"
```

之后再打开`~/.ssh/id_rsa.pub`拿到公钥内容，复制所有内容；

在github找到设置，add-sshkey，新建一个ssh-key。将内容粘贴进去。

之后再本地的git仓库下使用：

`ssh -T git@github.com`

进行验证。
