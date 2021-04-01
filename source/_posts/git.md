---
title: Git 基础
date: 2017-04-22 19:15:03
tags: 
	- Git
categories: Git
---

## Git 基础

### 常用 git 基础命令

克隆项目地址:`git clone [http://project/address]`

查看本地分支/详情:`git branch / git branch -v`

创建分支:`git branch branchName`

分支重命名:`git branch -m branchNameOld branchNameNew`

切换分支:`git checkout branchName`

创建并切换到分支:`git checkout -b branchName`
  
如果需要操作远程仓库的某个分支，可以用`git checkout -b branchName origin/branchName ` ，意思是基于远程仓库的 origin/branchName 分支的基础上创建本地的branchName分支，分支名最好一样，当然也可以不一样。在push的时候使用`git push origin HEAD:branchName`就可以了。

查看本地repo文件状态:`git status`

查看文件详细不同:`git diff`
<!--more-->
查看历史:`git log`

添加文件:`git add fileName  /  git add . `

commit文件:`git commit -m "注释信息"`

查看远程主机名:`git remote -v`

更新文件: `git pull origin master / git pull`,origin为远程名，master为分支名

提交文件: `git push orgin master / git push ` 格式:`git push <远程主机名> <本地分支名>:<远程分支名>`

查看远程分支:`git branch -r`

查看所有分支:`git branch -a`

常用的貌似就这些，但是远远不止这些，还有`git init`,`git rebase` 很大一部分命令，这么多我怎么记得住了？可以用给命令设置别名的方法:

1. 使用`alias`设置系统别名,每次会话结束后会失效.
	```
		如果要设置`gst`为`git status` 可以这样:
	
		设置别名:`alias gst='git status'`
		
		查看别名:`alias gst`
		
		覆盖别名:`alias  gst='覆盖后的命令'`
		
		删除别名:`unalias gst`
		
		如果需要每次都生效：mac写入到.bash_profile中就可以了```


2. git本身设置
	 ```
	 git config --global alias.st status
	 ```
	下次使用 `git st` 就可以
	
	

### github 远程仓库同步
有一种情况，如果你fork了别人的仓库到自己的github仓库，然后本地clone了自己github仓库里面的项目，这时候别人的仓库有了更新，而我们肯定也是要同步更新的，这个时候该怎么办？

解决方法:

1:增加fork的源仓库地址到本地:`git remote add remoteName https://github.com/remote/address`

2:查看分支信息:

```
git remote -v
origin	https://github.com/chenzhijun/java-core-learning-example.git (fetch)
origin	https://github.com/chenzhijun/java-core-learning-example.git (push)
upstream	https://github.com/ORIGINAL_OWNER/java-core-learning-example.git (fetch)
upstream	https://github.com/ORIGINAL_OWNER/java-core-learning-example.git (push)
```
这里的upstream就是原来的库，orgin指代的就是自己的github仓库了。 

3:同步远程库到本地:`git fetch upstream`

4:保证切换到master分支:`git checkout master`

5:合并更新到本地:`git merge upstream/master`

6:推送到自己的github仓库:`git push origin master`

这样github仓库和远程库是同步更新的了。

--
> 参考链接

1. [设置远程源](https://help.github.com/articles/configuring-a-remote-for-a-fork/)

2. [同步fork](https://help.github.com/articles/syncing-a-fork/)
