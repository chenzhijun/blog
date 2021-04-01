---
title: Golang 基础入门
copyright: true
date: 2019-07-14 12:55:21
tags: golang
categories: Go
---

# Golang 基础入门

了解任何事物首先得了解一下它的历史。golang 在2007年就开始开发，在09年开源，并且在12年发布了第一个稳定版本GO 1。今年19年，go这些年的发展是很快的，目前的版本是go 1.12.7，目前go的开发节奏是半年发布一个版本。并且golang承诺，go的更新时兼容之前的版本的。

## Golang 安装

在golang的官网[golang.org](https://golang.org/)下载最新版本的golang或者你需要的版本。接下来如果是windows的机器，那么只需要一路next就行了,之后在`c:\\go`文件夹下就可以了。如果是Linux系统，只需要将tar.gz包解压到自己工作的位置，然后将$GOLANG_PATH\bin加入到path就可以了。

<!--more-->

## Hello Go

开始一门新语言的探险时，大家的第一个demo肯定是“Hello World”。我们也开始第一个"Hello Go"吧：

```golang

package main

import "fmt"

func main(){
	fmt.Println("Hello,Go..")
}

```

将上面的内容保存为一个`main.go`文件，然后使用`go run main.go`。你就可以看到输出了，超级简单。

不得不说是真的简洁。看到上面的代码，我们开始进行一下基本golang开发介绍。

## main包main方法

golang中定义一个包的关键字就是`package`，程序的入口始终都是`main`包中的`main()`方法，程序始终都是从这里开始。
不像Java中的lang包不需要导入，golang中除了一些关键字，其它的不管是内部包，还是外部包都需要通过关键字`import`进行导入。

## GOROOT，GOPATH

golang 中有个路径非常重要：GOROOT 和 GOPATH ; 

GOROOT ： 其实就是golang的安装目录，当你用在控制台输入： `go env GOROOT`就能获取到值。

GOPATH ： 实际开发工作的目录。

GOROOT 就不多讲了，这里讲一下GOPATH；golang程序开发不像之前Java开发，Java开发，可能是项目名/src/路径-源码。而golang中，你的所有开发代码其实都是放在GOPATH下的，而且是在$GOPATH/src下。在GOPATH中定义了三个文件夹：`src`,`pkg`,`bin`。

1. pkg 存放项目编译时期的中间文件，比如:`.a`文件；
2. bin 目录存放项目的可执行文件，如果你把这个路径放到path目录下，那么就可以直接在控制台执行这里目录下的可执行文件；
3. src 源码存放路径记住，开发中一定要将代码放到这个路径下；比如Java开发者可能就不适应，因为之前的Java是`项目名/src`的方式，现在golang是`src/项目名`;

如果开发一个应用`myserver`,那么目录结构就是：`$GOPATH/src/myserver`; golang在编译的时候是会自动去找GOPATH下src目录的。这样你可以把所有相关联的项目放到一个GOPATH路径下，不同的项目之间可以隔开，也可把所有项目都放在一个GOPATH下，这样这个路径下的所有包，你都可以直接导入。当然你也可以照样像Java那样包路径结构，只是需要记住要将项目根路径指定为GOPATH，不然编译就会报错。

## golang的包

你可能注意到了我们上面`hello go`的第二行 `import "fmt"`；这里的作用就是导入fmt包，这个包在哪里了？golang寻找import包的优先级为：GOROOT-->GOPATH;如果找不到你也可以使用`go get`工具，它会帮你自动下载包;

项目开发中项目结构如下：

![2019-07-14-22-14-21](/images/qiniu/2019-07-14-22-14-21.png)

我们看一个包导入的例子：

![2019-07-14-22-42-50](/images/qiniu/2019-07-14-22-42-50.png)

我在项目中的handler下定一个了方法`Hello`，包名为`handler`;正常情况下，在main包中我们如果掉用它的话就是`handler.Hello()`;如下：

![2019-07-14-22-44-21](/images/qiniu/2019-07-14-22-44-21.png)

可以看到，main 方法中出了一场，报的是找不到handler包。这个时候要看一下我们的GOPATH了，之前我们也说了golang是先从goroot再从gopath去寻找包的。goroot，一般大家也不会将源码放那里，我们现在看下GOPATH的目录：

![2019-07-14-22-46-46](/images/qiniu/2019-07-14-22-46-46.png)

这下可以看到了吧。我们的项目路径是：`D:\workspace-paas\goinaction`;但是gopath的路径是`D:\Users\chenzj001\go`

我们可以知道GOPATH下是没有我们的handler包的。这个时候的修改方式有两个：

1. 将项目移到gopath目录的src目录下；
2. 将GOPATH设置为当前的工作目录；

如果采用1的话那么恭喜你，问题解决。如果采用2,哈，那么恭喜你，你肯定是Java派golang开发者，同道中人啊。你可能会想，我不是将GOPATH指到当前项目根路径了么？怎么还是导入不了。唉~。其实就像前面说的，go寻找包的时候都是去src目录下找的。所以你需要在项目路径下再建立一个src目录，再将源码放进去，就能解决异常了，采用2，就相当于是你要用Java开发的包路径结构。这种其实是不推荐的，入乡随俗还是独树一帜看你选择了。我是觉得按照大家的约定会比较好。

我们采用第一种方式，你看下图：

![2019-07-14-22-59-40](/images/qiniu/2019-07-14-22-59-40.png)

你可以看到上面的gopath。

> ps: 其实你是可以同时设置多个GOPATH的哦，可以自己动手尝试一下。


## GO 自带的工具

GO 自带了一些常用工具，使用`go --help`就能看到。这里介绍常用的几个：

1. `go build`,编译包
2. `go run`,编译运行
3. `go fmt`,golang内置的将代码格式化
4. `go doc`,golang的文档
5. `go test`,golang测试工具
6. `go get`,下载导入外部包

go 也可以编译成其它平台的可执行文件，比如window下编译成Linux下的执行文件[golang编译成Linux环境下的二进制文件](http://chenzhijun.me/2019/03/31/windows-compile-golang-to-linux-running-script/)。

go 的这些工具还是挺有用的，没事可以多用用。

这篇文章介绍了下go的发展史，也写下了`hello go`之后介绍了gopath，goroot，以及go自带的工具。其实golang其实是面向程序员友好的，对比Java动辄就是几十行代码，golang确实很简洁。不过对比Java来说，golang的生态其实感觉并没有Java那么的完善，另一方面，golang的包管理，我到现在还是有点。。还有一个就是在国内访问golang官网需要科学上网。这有点像什么了，我要推广给大家用，但是大家用起来又是各种阻碍，在已有的产品能实现需求的产品上，如果新产品不能提供很好的体验，其实要退光是有困难的。另一方面，golang其实面向的最多的感觉还是C/C++来转型。一门语言其实有它的优势，也肯定有它的不足。适合自己的业务场景，个人发展路径，取舍在于自己，我从一开始其实挺不喜欢golang的，但最近学会不排斥的心态去了解之后，感觉是真的爽。个人观点，不喜勿喷。一千个观众有一千个哈姆雷特。
