---
title: Linux 工具-tar
copyright: true
date: 2019-06-30 16:32:24
tags: tools
categories: Linux
---

# Linux 工具-tar

tar 命令应该是我们经常用的了，它主要的功能是用来对文件的解压缩操作。如果要看tar的具体操作可以使用:`man tar`，Linux的发行版本默认都会有 tar 命令

## 解压文件

tar 常用来解压`tar.gz`,`tar`的文件。使用的方式：`tar -zxvf xxx.tar.gz`

## 压缩文件

通常我们也会有需要将多个文件压缩成一个文件的需求，比如传输文件。这个时候我们就可以使用`tar -zvcf xxx.tar.gz file-dir-path1 file-dir-path2 file-dir-path-n` 其中`path1`,`path2`,`path-n`可以是多个或者单个。