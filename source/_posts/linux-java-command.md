---
title: 常用 Linux 命令
date: 2017-04-18 20:11:23
tags:
	- Java
	- Linux
---

## 常用 Linux 命令

### 1.查找文件

```
find / -name filename.txt 	根据名称查找/目录下的filename.txt文件。

find . -name "*.xml" 		递归查找所有的xml文件

find . -name "*.xml" |xargs grep "hello world" 递归查找所有文件内容中包含hello world的xml文件

grep -H 'spring' *.xml 		查找所以有的包含spring的xml文件

find ./ -size 0 | xargs rm -f & 删除文件大小为零的文件

ls -l | grep '.jar' 查找当前目录中的所有jar文件

grep 'test' d* 显示所有以d开头的文件中包含test的行。

grep 'test' aa bb cc 显示在aa，bb，cc文件中匹配test的行。

grep '[a-z]\{5\}' aa 显示所有包含每个字符串至少有5个连续小写字符的字符串的行。
```

### 2.查看一个程序是否运行
```
ps –ef|grep tomcat 				查看所有有关tomcat的进程

ps -ef|grep --color java 		高亮要查询的关键字
```
<!--more-->
### 3.终止线程
```
kill -9 pid
```

### 4.查看端口被占用的程序pid
```
lsof -i:8080    8080 为端口号
```

### 5.复制文件
```
	cp src taget
	cp -r src target
	scp srcFile remoteUserName@remoteIp:remoteFileAddress 将本地目录拷贝到远程目录 ； 加上-r参数 为文件夹拷贝
	scp remoteUserName@remoteIp:/path/filename /local/local_destination  将远程目录拷贝到本地目录 ；加上-r 为文件夹拷贝
```

### 6.创建和删除目录
```
	rmdir/mkdir
```

### 7.查看结尾1000行
```
	tail -1000f filename  查询动态日志文件的时候
```

### 8.查看端口占用情况
```
	netstat -tln | grep 8080 查看8080的使用情况
```

### 9.端口属于哪个程序
```
	lsof -i:8080
```

### 10.查看进程
```
ps aux|grep java 查看java进程
ps aux 查看所有进程
```

### 11.文件下载
```
wget http://file.tgz
curl http://file.tgz
```

### 12.远程登录
```
ssh userName@ip
```
### 13.打印信息
```
	echo $JAVA_HOME
```











