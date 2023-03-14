---
title: shell 脚本学习
copyright: true
date: 2018-05-13 18:02:31
tags: shell
categories: Linux
---

# shell 脚本学习

shell 脚本在linux上可以说是非常有用的一个工具，它就是linux命令的一个集合，所以写好shell脚本的关键一部分就是对linux命令比较熟悉。

本博客是在图书馆借阅shell相关脚本学习书记做的一个简单笔记。

## shell 脚本的创建与执行

一般我们在linux中创建的shell脚本都是以`.sh`为结尾的，这其实不是说一定要用sh结尾才行，只是大家约定习俗的一个习惯而已。
在shell文件的第一行，通常是`#!/bin/bash`，表示该脚本使用bash语法。`#`是shell脚本中的注释。通常自定义的脚本我通常习惯放在`/usr/local/bin`目录下,[关于bin,sbin,/usr/bin,/usr/local/bin等的区别](https://unix.stackexchange.com/questions/8656/usr-bin-vs-usr-local-bin-on-linux)

现在我们创建第一个脚本`first.sh`:

```shell
#!/bin/bash
echo "Hello,world"
```
<!--more-->
脚本创建好之后有两种运行方式:
1. 使用bash执行：`bash first.sh`
2. 直接执行：先让first.sh变成可执行脚本`chmod +x first.sh`,然后再直接运行`./first.sh`

脚本运行过程中可以使用`bash -x first.sh`，这样可以看到脚本的执行过程。

## shell 脚本的变量

### 脚本预定义变量

shell 预先设置的几个变量值，比如`optin.sh`：

```shell
#!/bin/bash
echo $0 $1 $2
```

如果执行：`./option.sh 1 2`,输出为：`option.sh 1 2`

也就是执行脚本的时候后面的参数有几个，在脚本中直接使用`$位置`就可以使用这些变量值，0位置是脚本的名称。

### 自定义变量

在脚本中自己定义的变量，`变量名=变量值`，在脚本中使用的方式为`$变量名`。

```shell
#!/bin/bash
num=1
echo "$num"
```

`num=1` 中`num`和`=`之间不要有空格,不要有空格，不要有空格。

### 控制台变量值传递

控制台交互性，读取控制台输入的数，然后传递给脚本中，如：

```shell
#!/bin/bash

read -p "Please input a number: " x
read -p "Please input another number: " y

sum=$[$x+$y]

echo "the sum of the two numbers is : $sum"

##chen@ubuntu:~/shell-learn$ bash read.sh 
#Please input a number: 1
#Please input another number: 3
#the sum of the two numbers is : 4

```

## 数值运算

简单的加减乘除的运算，使用`[]`前面需要加上`$`，如：

```shell
#!/bin/bash
#!/bin/bash

a=1
b=2
sum=$[$a+$b]

echo "$a+$b=$sum"

echo ================

sum2=$[$a+$b]
echo "$a+$b=$sum2"


## sh sum.sh 1+2=$[1+2]
## bash sum.sh 1+2=3
## ./sum.sh  1+2=3
## 这三种方式执行结果不一样
```

## if-else 条件逻辑语句

对于if-else大家在任何一门语言中都是必备的，shell中的if-else格式如下：

### 常用的数值判断

if-else常用的根据数字来判断的相关实例：

```shell
# 单独if语句
if 判断语句; then
    command
fi

# if-else 语句
if 判断语句; then
    command
else
    command
fi

#if-elseif-else 语句
if 判断语句; then 
    command
elif 判断语句2; then
    command
else
    command
fi
```

如：

```shell
#!/bin/bash

num=$1

if ((num<60));then
	echo "you don't pass the exam; your score is $num"
elif ((num>=60&&num<80));then
	echo "good job, your score is $num"
else
	echo "great job, your score is $num"
fi

if ((num==60));then 
	echo "ok"
fi

if ((num!=50));then
	echo "not equal 50"
fi
```

执行的时候：`./if.sh 89`。

数值大小的判断除了有`(())`之外，还可以使用`[]`，如果使用`[]`，那么就不能使用`> , < , =`这些符号，要使用:
小于：`-lt`，大于：`-gt`，小于或等于：`-le`，大于或等于：`-ge`，等于：`-eq`，不等于：`-ne`，如：

```shell
#!/bin/bash
a=10
if [ $a -lt 5 ];then echo ok;fi
if [ $a -gt 5 ];then echo gt;fi
if [ $a -ge 5 ];then echo ge;fi
if [ $a -eq 5 ];then echo eq;fi
if [ $a -ne 5 ];then echo ne;fi
```

### if中常用的跟文档相关的判断

if 语句中可以使用一些内置的跟文件判断相关的参数。

```shell
#!/bin/bash

# if判断跟文件相关的一些参数

# -e 判断文件或目录是否存在
# -d 判断是不是目录，以及是否存在
# -f 判断是不是普通文件，以及是否存在
# -r 判断是否有读权限
# -w 判断是否有写权限
# -x 判断是否可执行

if [ -e /home/ ]; then echo ok ;fi

if [ -f /home/ ]; then echo ok ;fi

if [ -f /home/chen/shell-learn/if1.sh ]; then echo ok ;fi

```

## case 逻辑判断

java中有if之外还有switch，shell中除了if之外还有case,如果总结一下啊，可以发现if中的结束用fi，case中结束用esac，刚好是倒过来的。

```shell
# case语句，value可以任意个，*指代其它值。类似java switch default
 case 变量 in
 value1)
 	command
 	;;
 value2)
 	command
 	;;
 value3)
 	command
 	;;
 *)
 	command
 	;;
 esac
```

实例脚本，判断奇偶数字：

```shell
#!/bin/bash
num=$1
n=$[$num%2]

case $n in
1)
	echo "奇数"
	;;
0)
	echo "偶数"
	;;
*)
	echo "其它的值"
	;;
esac

```

之前跟同事学的一个很有用的基础脚本：[地址](http://chenzhijun.top/2018/05/10/a-easy-script/)

## shell中的循环：for，while

常用到的循环就是for和while了，

for的使用方式：

```shell
#!/bin/bash

# 脚本循环
#for 变量名 in 循环的条件; do
#	command
#done

for i in 1 2 3 4 5 6; do
	echo $i
done

for filename in `ls`; do
	echo $filename
done

```

while 的使用方式：

```shell
#!/bin/bash

# while 条件; do
# 	command
# done

num=5 

while [ $num -ge 1 ]; do
	echo $num
	num=$[ $num-1 ]
done

```

while 中如果是`while :; do`那么就是死循环了，有些地方可以这样使用。


## shell 函数

shell的函数一定要在被调用之前声明，生命函数的关键字是`function`,函数的参数个数需要在调用的时候指定，如`func.sh`：

```shell
#!/bin/bash

function sum()
{	
	sum=$[$1+$2]
	echo $sum
	echo "第三个变量：" $3
}

sum $1 $2

echo "======================="

sum $1 $2 $3
```

调用的方式`./func.sh 1 2 3` 输出结果为：
![2018-05-13-18-56-53](/images/qiniu/2018-05-13-18-56-53.png)

可以看到第一个函数里面的`$1 $2 $3`跟shell脚本预设的变量是不相关的，函数里面能用的参数都是`sum`传递进去的。

> ps: 如果每次写完脚本都需要使用`chmod +x xxx.sh`这样也挺烦的，不如写个脚本吧，`x.sh`。

```shell
chmod +x *.sh
```
这样就可以使用`./x.sh`，直接执行脚本就好了。