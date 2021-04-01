---
title: byte 在golang和Java中的区别
copyright: true
date: 2019-03-31 21:45:50
tags: byte
categories: Java
---

# byte 在golang和Java中的区别

最近做一个项目，想用到md5，做一次数据的校验。因为是两个系统，一个golang开发，一个java开发。首先用Java生成md5值，然后传给golang，发现原始数据一致，但是生成的md5值却不一致。

深究其原因，最终发现是golang和java中对于byte的定义一个是无符号的，一个是有符号的，所以两者最后生成的md5值不一致。现在我发现有的时候真的需要注意到一些比较基础的东西，不然就会成为面向api的工程师。