---
title: shell 相关基本操作
tags: shell
copyright: true
date: 2019-09-07 12:50:53
categories: Linux
---


# shell 相关基本操作

## shell 基础

shell中的脚本通常就是控制台中的语句，将这些语句结合到一个文件中，就组成了脚本。脚本的第一行通常是`#!/bin/bash`开通，这行的作用是指定用哪个shell。可以`cat /etc/shells`查看当前操作系统支持哪些shell。

在控制台中我们使用 `;` 隔开多条语句，在shell脚本中则是一行命令独立一行。通常我们会以`*.sh`结尾来作为一个脚本名称，另外会将其权限设置为可执行权限：`chmod u+x *.sh`。

<!--more-->


## 管道与重定向

管道的符号是：`|`，作用是将前一个命令的操作结果传给第二个命令：`ps | cat`，`echo 123 | ps`


重定向的符号:

1: 输入重定向：`<`;
2：输出重定向`>`,`>>`,`2>`,`&>`;

输入输出组合使用：

```shell
cat > /path/to/a/file << EOF
I am test
EOF
```

`>` : 先会清空文件然后再输出到文件

`>>` ：追加到文件中，会在最后一行添加

`2>` ：如果出现异常，将异常结果输出到文件中

![2019-09-07-11-36-12](/images/qiniu/2019-09-07-11-36-12.png)

`&>` : 不管结果对错都输入到文件中

![2019-09-07-11-41-02](/images/qiniu/2019-09-07-11-41-02.png)


## shell 脚本执行的几种方式

`bash xxx.sh` : 开了一个子进程bash执行脚本，可以不需要shell文件有可执行权限，就可以执行脚本；

`./xxx.sh` ：同bash，不过需要脚本有可执行权限；

`source xxx.sh` ：在当前shell内去执行脚本，脚本可以没有可执行权限；

`. xxx.sh` ：source的缩写；

![2019-09-07-11-53-42](/images/qiniu/2019-09-07-11-53-42.png)

## 变量定义与使用

```shell
NAME=chenzhijun
echo $NAME
```

上面中`=`两边不能有空格，不然的话会将 NAME 识别成一个命令来报错。使用变量 NAME 的时候只需要带上`$`即可。

### 环境变量

`env` `set` 获取环境变量；

`$PS1`可以修改总端显示

### 预定义变量

`$?` : 使用`echo $?`可以获取上一条命令执行的结果，成功为0，失败为1

`$$` ：获取当前的pid

`$0` ：当前进程名称

![2019-09-07-12-15-44](/images/qiniu/2019-09-07-12-15-44.png)

执行的方式不同，`$0`的值也不同。

### 位置变量

```shell

#/bin/bash
# $1 $2 ...$9 ${10}

pos1=$1
pos2=${2-_} # 如果$2为空值，用`_`代替

```

### 环境变量配置文件

```shell

/etc/profile
/etc/profile.d/

~/.bash_profile
~/.bashrc

/etc/bashrc

```

`/etc/profile*` 是所有用户通用的。`~/.bash*`是用户特有的。 

用户登录分为`login shell`，`nologin shell` 。如果是`su - user`是`login-shell` ，如果是`su user`是`nologin shell`

如果在每个文件第一行加入`echo xxx`，可以看到如下加载顺序：

![2019-09-07-12-34-30](/images/qiniu/2019-09-07-12-34-30.png)

要注意作用域的问题，shell执行的时候通常是一个subshell，也就是一个子进程shell，要注意变量是否可以传递过去。一般可以用`source /etc/profile`来重加载；也使用export也可以将当前shell的变量传递subshell中。

![2019-09-07-12-39-06](/images/qiniu/2019-09-07-12-39-06.png)

上面的方式是在`/etc/profile`文件最后面增加了`aaa`变量，使用source可以加载到当前shell中，使用export可以在subshell（bash命令）也同样获取到`aaa`的值。


## 常用的一些脚本命令

1. 找出dir目录下文件名带 `xxx` 的文件 : `find /dir -name *xxx*` ; 
2. 找出dir目录下文件中带 `xxx` 内容的文件 ：`grep -r "xxx" /dir` ;
3. 找出dir目录下文件名带 `aaa` 的文件并且文件路径中有 `bbb` 然后查找这些文件中含有 `ccc` 的文件，替换这些文件中的 `ddd` 为 `eee`: `find /dir -name "*aaa*"|grep 'bbb'|xargs grep 'ccc' -l|xargs sed -i 's/ddd/eee/g'` ;
`grep -l` 为输出全路径。


## 参考资料

[github shell 大全](https://github.com/dylanaraps/pure-bash-bible)