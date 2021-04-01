---
title: gitlab 安装与配置（一）
copyright: true
date: 2018-03-30 23:28:38
tags: gitlab
categories: git
---

很多公司都是使用的 svn 来管理代码，其实我感觉 git 肯定是未来的潮流。最近闲暇时间，在公司搭建了一个 GitLab ，正好可以记录一下。

> gitlab 的搭建环境推荐使用 >=4G 内存



## 使用 Docker 部署 gitlab

首先需要安装 Docker。推荐使用docker官网的安装方式：[Centos 安装docker](https://docs.docker.com/install/linux/docker-ce/centos/)，如果访问docker官网过于太慢，可以使用DaoCloud下载docker：[daocloud 安装docker](https://download.daocloud.io/Docker_Mirror/Docker)。有时候下载镜像，有一些比较显而易见的原因，所以可以使用[docker镜像加速器](https://www.daocloud.io/mirror#accelerator-doc)。

安装docker之后，下载镜像`gitlab/gitlab-ce:latest`，之后参照[安装文档](https://docs.gitlab.com/omnibus/docker/README.html#run-the-image)。

```shell
sudo docker run --detach \
    --hostname gitlab.example.com \
    --publish 443:443 --publish 80:80 --publish 22:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```
一个gitlab就搭建好了。**默认的帐号为`root`**。

<!--more-->

### 配置邮件服务

在gitlab的配置目录有个文件：`/etc/gitlab/gitlab.rb`，编辑这个文件。

```shell
sudo docker exec -it gitlab vi /etc/gitlab/gitlab.rb
```

如果你是用的docker方式，看上面的命令，可以编辑本地目录`/srv/gitlab/config/gitlab.rb`也是可以的，不过需要在docker容器里面进行配置重加载。

安装邮件的文档：[配置smtp](https://docs.gitlab.com/omnibus/settings/smtp.html#doc-nav)

贴一份在实际中使用的邮件配置：

```s
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'name-gitlab@company.com'
gitlab_rails['gitlab_email_display_name'] = 'gitlab邮件通知'
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.mxhichina.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "name-gitlab@company.com"
gitlab_rails['smtp_password'] = "email-password"
#gitlab_rails['smtp_domain'] = "ap-ec.cn"
gitlab_rails['smtp_authentication'] = "login" # 认证的方式
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true

user['git_user_email'] = "name-gitlab@company.com"
```

安装完邮件后进入到容器中：`docker exec -ti gitlab bash`，之后使用`gitlab-ctl reconfigure`进行配置重加载，加载完成后需要进行测试：`gitlab-rails console`

```shell
Notify.test_email('test-email-name@company.com', 'Message Subject', 'Message Body').deliver_now
```

输入上面的命令如果有新邮件收到，那么邮箱smtp就配置成功。

> 如果smtp配置成功，但是注册的时候还是没有邮件发送通知，那么重启一下容器。

