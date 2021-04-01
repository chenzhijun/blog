---
title: 使用Travis-CI持续构建Hexo博客
date: 2017-12-19 14:05:44
tags:
	- Hexo
    - Blog
categories: Hexo
---

## 使用Travis-CI持续构建Hexo博客

我是用的 Hexo+Github Page 来构建自己的博客。在 Github 创建`your_github_name.github.io`仓库的时候，可以直接使用`your_github_name.github.io`作为你的博客域名站点。详细的话，可以自己去 google 下使用 Hexo搭建个人博客。

写本文的原因是因为，每次我在写完一篇总结，提交了push之后。如果要更新到博客上，需要经历的过程就是：

```shell
    # 提交源文件到仓库
    git add .
    git commit -m "xxx"
    git push origin master

    # 发布到博客站点
    hexo clean
    hexo d -g
    # 这里还需要输入github账号，密码。
```

我觉得如果提交源文件push之后能够直接发布到博客站点就好了。在网上搜了一圈，很多人的资料还是有些缺陷，自己踩了一路坑，所以有了本篇记录。

### 环境介绍

先介绍下我的博客环境，我用了两个repo:
一个是存博客源码的`blog.git`,
另一个也就是用来做站点发布的`chenzhijun.github.com.git`。

> 网上很多人喜欢用一个库，然后切换分支的做法。之前我也弄过，不过后来发现我经常弄错分支。反正repo不要钱，就无所谓，分开吧。其实就是懒。总之，自己爽就好。
<!--more-->

### 实际操作

其实就是将生成的目录public下的所有文件当作了chenzhijun.github.com.git库下面的文件。

1. 生成一个[Personal access tokens](https://github.com/settings/tokens).Token记得勾选如下权限，然后copy保存：

![2017-12-19-15-21-03](/images/qiniu/2017-12-19-15-21-03.png)

2. 注册Travis-CI账号：[https://travis-ci.org](https://travis-ci.org/)。我是直接使用github登陆的，方便。

3. 激活需要进行CI的仓库：

![2017-12-19-15-25-02](/images/qiniu/2017-12-19-15-25-02.png)

点击仓库进去做配置，找到设置的地址：

![2017-12-19-15-25-48](/images/qiniu/2017-12-19-15-25-48.png)

配置环境变量等：

![2017-12-19-15-29-15](/images/qiniu/2017-12-19-15-29-15.png)

4. 之后在博客根目录创建`.travis.yml`文件，我的文件内容为：

```yml
language: node_js # 设置语言

node_js: stable # 设置相应版本

cache:
    apt: true
    directories:
        - node_modules # 设置缓存，传说会在构建的时候快一些

before_install:
    - npm install hexo-cli -g

install:
    - npm install # 安装hexo及插件

script:
    - hexo clean # 清除
    - hexo g # 生成

after_script:
    - git clone ${GH_REF} pub_web # 因为我有两个仓库，先将发布服务的仓库clone下来，
    - cp -rf public/* pub_web/ # 将源博客仓库(blog.git)目录下的public文件夹下的文件复制到发布服务的仓库(chenzhijun.github.com.git)中
    - cd pub_web # 进入到git仓库
    - git config user.name "username"
    - git config user.email "email@address.com"
    - git add .
    - git commit -am "Travis CI Auto Builder :$(date '+%Y-%m-%d %H:%M:%S' -d '+8 hour')" # 零时区，+8小时
    - git push origin master 
branches:
    only:
        - master #只监测master分支,这是我自己的博客，所以就用的master分支了。
env:
    global:
        - GH_REF: https://yourname:${GITHUB_TOKEN}@github.com/yourname/your.blog.address.git #设置GH_REF，注意更改yourname,GITHUB_TOKEN:就是我们在travis-ci仓库中配置的环境变量
```

5. 在`_config.yml`中加入(这里如果是用hexo，应该一开始就会弄好了)：

```yml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/yourname/yourname.github.com.git
  branch: master
```

到这里，我们一个简单的ci就弄好了,你可以试着提交一个commit，然后push到你的仓库之后在travis-ci里面就能看到build日志了。

![2017-12-19-16-17-25](/images/qiniu/2017-12-19-16-17-25.png)
![2017-12-19-16-17-42](/images/qiniu/2017-12-19-16-17-42.png)

其实简单的原理就是：

1. 向仓库`blog.git`提交commit；
2. travis-ci 自动构建`blog.git`,根据`.travis.yml`的配置执行；
3. 运行`hexo g`之后，`public`目录下文件更新；
4. 克隆`chenzhijun.github.com.git`仓库，将其命名为别名`pub_web`，将public下的文件复制到`pub_web`；
5. 将pub_web目录下的文件提交commit；
6. push最新的文件到github。

> 特别注意 personal token只在仓库的https协议下有效，官网说的：[Creating a personal access token for the command line](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)

这样站点就更新了，如果你是在一个仓库下多个分支，我想应该也容易了。自己动手，丰衣足食。


相关参考：

[使用Travis CI自动部署Hexo博客](http://www.itfanr.cc/2017/08/09/using-travis-ci-automatic-deploy-hexo-blogs/)

[使用 Travis-CI 来自动化部署 Hexo](http://zhzhou.me/2017/02/20/auto-deploy-hexo-on-travis-ci/)

[基于 Hexo 的全自动博客构建部署系统](http://kchen.cc/2016/11/12/hexo-instructions/)