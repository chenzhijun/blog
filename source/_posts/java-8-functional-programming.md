---
title: Java 8 函数式编程-前言
date: 2017-11-21 14:35:13
tags:
    -Java
    -Lambda
categories: Java
---

## Java 8 函数式编程-前言

>从 Java 8 发布到现在已经过去很久了，现在 Java 9 也都已经发布了。国人的习惯总是你发布 8 的时候，我用 7 。你发布 9 的时候，我想 Java 8 应该是普及的时候了。那么正当时，还不来普及下 Java 8 的一个大特性，那就有点说不过去了。

----------------

------------

### 引言 - 什么是函数式编程

函数式编程（Funtional Programming，以下简称 fp ），我想从 Java 8 发布之后大家对这个名词都不陌生，它有很多好处，减少代码，增加可读性等等。但是什么是函数式编程了？它和我们普通的 OOP 又有什么不同？

先来看一份常规打招呼的代码，在 Java7 之前我们大多是这么玩的：
<!--more-->
```java
 /**
 * 打招呼
 */
    public static  String greetJdk7(List<String> names) {
        String greeting = "Hello ";
        for(String name : names) {
            greeting += name + ",";
        }
        greeting += "!";
        return greeting;
    }
```

Java8 之后如果用函数式编程的方式改写代码：

```java
    public static String greetJdk8(List<String> names) {
        String greeting = names
                .stream()
                .map(name -> name + ",")
                .reduce("Hello ",
                        (hello, name) -> hello + name);
        return greeting + "!";
    }
```

两者对比起来我们会发现，干净利落，但是看不懂，没关系，之后我会告诉你上面用 Java8 写的代码的含义，并且你也能自己写 Functinal Program。 

在开始之前，我想提前和你分享一些知识：

1. 在fp中使用的**所有变量**都是`final`的，final的含义意味着，在fp中你不能对它进行改变，总之`final`的使用，我想你是知道的。

非法实例：

```java
    String str = "test";

    List<String> aList =new ArrayList();

    aList.forEach(obj->{
         str="bbb";// 这样是不允许的，因为str在这里相当于final
    })

```

2. 使用局部参数代替全局变量,比如说：

```java
public class Utils {

    private static Time time;

    public static String currTime() {
        return time.getTime().toString();
    }

}
```

替换成

```java
public class Utils {

    public static String currTime(Time time) {
        return fixedTime.now().toString();
    }

}
```

3. 将方法作为参数使用：

```java
public List<Integer> addOne(List<Integer> numbers) {
    List<Integer> plusOne = new LinkedList<>();

    for(Integer number : numbers) {
        plusOne.add(number + 1);
    }

    return plusOne;
}
```

转换成

```java
    public List<Integer> addOne(List<Integer> numbers) {
        return numbers
                .stream()
                .map(number -> number + 1)
                .collect(Collectors.toList());
    }
```

<!--
### 函数对象

Java 8 中，函数成为了“第一等公民”，这就意味着，我们可以将函数作为其他函数(俗称方法)的参数，或者将函数作为返回参数，或者像对象一样定义一个函数来使用。

未完待续
-->
