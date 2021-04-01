---
title: 从 SVN 迁移到 Git （二）
copyright: true
date: 2018-04-02 11:55:10
tags: 
    - gitlab
    - svn
categories: git
---


从 SVN 导入到 GitLab 仓库中我们只需要下载一个 `git-svn` 的工具，如果是windows版本的git工具，应该是内置了的；Linux 下使用`yum install git-svn`。一些操作可以参考git官网的两篇文章：[Git-与其他系统-Git-与-Subversion](https://git-scm.com/book/zh/v1/Git-%E4%B8%8E%E5%85%B6%E4%BB%96%E7%B3%BB%E7%BB%9F-Git-%E4%B8%8E-Subversion)和[Git-与其他系统-迁移到-Git](https://git-scm.com/book/zh/v1/Git-%E4%B8%8E%E5%85%B6%E4%BB%96%E7%B3%BB%E7%BB%9F-%E8%BF%81%E7%A7%BB%E5%88%B0-Git)。

从实际的svn迁移到git中有两种方式:

## 1) 最简单最直接的方式，从svn拷下代码，然后上传到gitlab仓库中。

先将svn上的代码拷下来，`git svn clone https://192.168.1.12/svn/trade/App_cn/Src/_CJ208_Noe`。

![2018-04-02-11-23-02](/images/qiniu/2018-04-02-11-23-02.png)

然后使用git将其推送到gitlab仓库。

```shell
git remote add origin http://gitlab.xxx.com/xxxgroup/_CJ208_Noe.git
```
<!--more-->

推送过程记得使用`root`管理：

![2018-04-02-11-28-59](/images/qiniu/2018-04-02-11-28-59.png)

可以在项目提交历史看到相应的提交记录。
![2018-04-02-11-29-41](/images/qiniu/2018-04-02-11-29-41.png)

这种方式提交的话，可以看到提交人的信息跟gilab是没有绑定的，这样可能就不太方便我们去查找某人，尤其是svn账号不规范的情况。

![2018-04-02-11-30-49](/images/qiniu/2018-04-02-11-30-49.png)

上图就是使用`czj`这种缩写，有些svn账号又是全名。有可能是因为历史原因，不过如果想直接从svn导入到git，这种方式是最方便的。当然可能在历史的提交记录里面有一些杂的信息。比如：`git-svn-id:XXXX`这些，进入到项目里面使用`git log`可以看到`commit`信息：

![2018-04-02-11-48-08](/images/qiniu/2018-04-02-11-48-08.png)

## 2) 将用户名和账号对应起来，然后再上传代码。不过在上传之前我们先过滤一下：

a) 先将用户名称过程，可以使用下面两个方式将svn提交的用户名存放到文本文件中：

直接使用svn地址：

```shell
svn log https://192.168.7.2/svn/trade/App_cncsen/Src/CJ325_Integral --xml | grep -P "^<author" | sort -u | \
      perl -pe 's/<author>(.*?)<\/author>/$1 = /' > users.txt
```

将代码从svn拷下来之后，进入到代码根目录里面使用：

```shell
svn log ^/ --xml | grep -P "^<author" | sort -u | \
      perl -pe 's/<author>(.*?)<\/author>/$1 = /' > users.txt
```

在`users.txt`中我们可以看到提交人的账号信息：

![2018-04-02-11-40-24](/images/qiniu/2018-04-02-11-40-24.png)

之后我们将其匹配到gitlab中的用户名：

![2018-04-02-11-43-03](/images/qiniu/2018-04-02-11-43-03.png)


然后使用 `git svn clone` 将代码拷下来，不过这次拷贝我们加入用户的信息：

```
git svn clone https://192.168.7.246/svn/trade/App_cncsen/Src/CJ325_Integral  --authors-file=users.txt --no-metadata
```

之后再进行一些过滤:

过滤svn中tag分支：

```
git for-each-ref refs/remotes/tags | cut -d / -f 4- | grep -v @ | while read tagname; do git tag "$tagname" "tags/$tagname"; git branch -r -d "tags/$tagname"; done

```

然后将 `refs/remotes` 下面剩下的索引变成本地分支:

```
git for-each-ref refs/remotes | cut -d / -f 3- | grep -v @ | while read branchname; do git branch "$branchname" "refs/remotes/$branchname"; git branch -r -d "$branchname"; done
```

![2018-04-02-11-50-02](/images/qiniu/2018-04-02-11-50-02.png)


这样过滤之后现在再发布到远端，下图可以看到，现在的历史中干净很多，而且名字也对应上了。

![2018-04-02-11-53-25](/images/qiniu/2018-04-02-11-53-25.png)

现在就可以开心的使用git了。
