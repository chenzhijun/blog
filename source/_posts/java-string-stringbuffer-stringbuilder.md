---
title: 查漏补缺 Java 系列 - String，StringBuffer，StringBuilder有什么区别
copyright: true
date: 2021-03-18 23:30:30
tags: Java
categories: Java
---

# 查漏补缺 Java 系列 - String，StringBuffer，StringBuilder有什么区别

## String，StringBuffer，StringBuilder 他们三者的优势和劣势？

一般来说，Java中String是用来对字符串进行操作的类，是一个 Immutable 类，它是被声明为final class，所有的属性也是final的；String中的所有跟字符串修改的方法都是新建了一个String，就会产生很多额外的String对象；

StringBuffer是一个线程安全的类，它的所有方法都加上了synchronized; StringBuffer还有个优点就是它在执行append的操作的时候将新的字符串插入到原来的串中，可能是末尾，也可能是中间位置；

StringBuilder 是 Java 1.5 中新增的，它可以看成是StringBuffer的非线程安全实现版本。
<!--more-->
## String，StringBuffer，StringBuilder 三者的使用场景？

其实在jdk9开始，如果我们javap String相关的源码会发现，不再有之前的StringBuilder出现了。这是因为在新的jdk9中，采用了一种新的方式：COMPACT_STRINGS 。底层的数据结构用的是 一个bytes数组+coder的方式组建String的内容，用coder来判断是用LATIN1,UTF16。那么很明显了。
如果你在jdk8里面，需要线程同步安全的化，可以使用StringBuffer；如果涉及到大量的String操作，可以考虑使用StringBuilder；但是我认为这个还是要根据编码来，编码首先是给人看的，一定要保证可读性。比如下列的：

```java
        StringBuilder sb = new StringBuilder();
        sb.append("abc").append("def");//如果很长了？

        String abcdeft="abc"+"def";
```

## 三者的关键组成部分和关键点？

> ps: jdk8中用的是数组char；jdk9+中用的是数组byte；


## 三者的底层原理和关键实践

