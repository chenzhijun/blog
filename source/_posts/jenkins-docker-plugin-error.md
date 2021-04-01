---
title: jenkins-docker-plugin-error
copyright: true
date: 2019-07-24 18:23:51
tags:
categories:
---

# Jenkins Docker 模块无法切换Registry credentials

问题产生，一次公司Jenkins迁移过程中，发现一个问题，dockers模块选择不同用户的 Registry credentials 但就是无法推送到私有仓库上去。

![2019-07-24-18-28-43](/images/qiniu/2019-07-24-18-28-43.png)

像上图一样，选择了用户，deployop 或者其它，但是Jenkins push的时候就是报没有权限：

![2019-07-24-18-30-05](/images/qiniu/2019-07-24-18-30-05.png)

就算我选的的是harbor的admin账号也没用。

后来想到jenkins的机器我是都手动执行过`docker login`的，不知道会不会有影响，于是手动去机器上docker build --> docker push 发现结果一样。也是没有权限。然后用docker login harbor.xxx.com 查看当前用户却不是admin账户，这就有点奇怪了，我当时在jenkins模块docker中选的就是admin啊。为什么docker 还是用之前的权限较小的账户了？然后我在~/.docker/config.json 查看到确实有两个账户，这个可以说明其实Docker 模块的Registry credentials其实际是起到了作用的。应该是我们一开始使用了docker login 所以造成了有"默认账户"。jenkins应该就是使用了默认账户才导致这个问题的。

![2019-07-24-18-35-44](/images/qiniu/2019-07-24-18-35-44.png)

解决的方式很简单，一个是删除`~/.docker`这个文件夹；一个是使用`docker logout harbor.xxx.com`就可以了。再用Jenkins构建的时候就可以看到该文件内容：

![2019-07-24-18-39-09](/images/qiniu/2019-07-24-18-39-09.png) 

如果你是删除了`~/.docker`文件夹，然后重启docker，那么你再次构建的时候应该会看到`~/.docker`文件夹变空了。但是在用户目录多出一个`~/.dockercfg`文件。里面的内容和config.json一致。如果你是使用docker logout，那么就还是会在`~/.docker/config.json`文件中看到你每次选择的用户。

![2019-07-24-18-43-39](/images/qiniu/2019-07-24-18-43-39.png)



> ps 如果想在jenkins启动的时候就指定其工作目录，可以这样设置`export JENKINS_HOME=/data/jenkins/.jenkins` 