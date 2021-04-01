---
title: 常用的 Linux 基础命令
date: 2017-04-18 19:15:03
tags: 
	- Linux
categories: Linux
---

### 常用 Linux 命令

`man`: 查询帮助

```
	man info
	man ls
```

`ls`: 列出目录

```
	ls 目录1 目录2  -> 同时列出目录1，2的文件夹下的文件
	ls -R 目录名    -> 递归列出目录下的所有目录和文件
```

`find`: 查找文件

```
	find 目录 -name 文件名 
	find ./blog/chenzhijun.github.com -name "*.md" | xargs grep "Life"
	| xargs 传递命令查找出./blog/chenzhijun.github.com目录下所有md 为结尾的文件，
	并且在在文件中查找内容包含 Life 字符串的内容```

<!--more-->
`grep`: 根据条件查找文件内容，配合find找更牛逼


```
	grep PATTERN filename -> 在文件filename中找到pattern ```


`pwd`: 当前工作目录的全路径

`rm`: 移除文件，主要参数-rf ，强制递归删除

`mv`: 移动文件

`cat`: 查看全部文件内容

`more`: 查看文件内容，逐行显示。

```
	more +100 filename
```

`less`: 查看文件内容，上下滚动查看

```
	less +100 filename
```

`vim/vi`:编辑文件，如果不存在就创建

`ping`: 测试网络连通

```
	ping www.baidu.com
```

`tar`: 文件解压缩

```
-c 创建归档
-x 解压归档
-v 显示处理过程
-f 目标文件，后面必须跟目标文件
-j 调用bzip2 进行解压缩
-z 调用gzip 进行解压缩
-t 列出归档中的文件
	
tar -cvf filename.tar .       ### 将当前目录所有文件归档，但不压缩，注意后面有个’.‘ ，不可省略，代表当前目录的意思 
tar -xvf filename.tar         ### 解压 filename.tar 到当前文件夹
tar -cvjf filename.tar.bz2 .  ### 使用 bzip2 压缩
tar -xvjf  filename.tar.bz2   ### 解压 filename.tar.bz2 到当前文件夹
tar -cvzf filename.tar.gz     ### 使用 gzip  压缩
tar -xvzf filename.tar.gz     ### 解压 filename.tar.gz 到当前文件夹
tar -tf   filename            ### 只查看 filename 归档中的文件，不解压

```

`ln`: 两个文件中创建链接，硬链接，软链接

```
ln source dest       ### 为 source 创建一个名为 dest 的硬链接

ln -s source dest    ### 为 source 创建一个名为 dest 的软链接

```

`chmod`: 改变文件权限，读，写，执行；

```
ls -al 可以查看文件的详情，其中 所有者 、 用户组 、 其他都占3个

-rwxr--r-- 1 locez users   154 Aug 30 18:09 filename

r=read,w=write,x=execute
	
	
 chmod +x filename        ### 为 user ，group ，others 添加执行权限
 chmod -x filename        ### 取消 user ， group ，others 的执行权限
 chmod +w filename        ### 为 user 添加写入权限
 chmod ugo=rwx filename   ### 设置 user ，group ，others 具有 读取、写入、执行权限
 chmod ug=rw filename     ### 设置 user ，group 添加 读取、写入权限
 chmod ugo=--- filename   ### 取消所有权限
 
 rwx对应111，如果赋予rwx三个权限，就是7，如果给所有者，用户组，其它都赋予rwx的权限，那么就是777
```

`wget`: 下载工具

```
wget -O newname.md https://github.com/LCTT/TranslateProject/blob/master/README.md     ### 下载 README 文件并重命名为 newname.md
wget -c url     ### 下载 url 并开启断点续传
```














