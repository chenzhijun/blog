---
title: Java 基础数据类型
date: 2017-03-21 23:16:13
tags:
	- Java
categories: Java
---

## Java 基础数据类型

> Java 拥有着八大基本数据类型：byte,short,int,long,float,double,boolean,char


### byte,Byte
了解byte之前我们先了解一下 "字节，位"。字节的单位是Byte，位的单位是bit。一个字节(Byte)=8个位(bit)。
在Java里面byte数据类型占用8位，它是带符号位的，怎么明白符号位的意思？一个位代表一个1或者一个0。8位带符号的意思是第一个位表明为符号位，如：0111111，符号位为0代表+，即最大值为127；如果是 11111111,最小值为-128。byte的包装类型为Byte,初始值为0。
<!--more-->
### short,Short
short 在Java里面占用16位，也是带符号位的。最小值为-32768最大值为32767。short的包装类为Short，初始值为0。

### int,Integer
默认情况下占用32位带符号位。从$-2^(31)$-$2^(31)-1$。在官网下得知，在Java8以及之后的版本中，也支持无符号32位int类型，数值从$0$-$2^(32)-1$。int的包装类型为Integer，初始值为0。

### long,Long
长整形数据类型占用64位带符号位。从$-2^(63)$-$2^(63)-1$。使用长整形的主要原因是在int不满足需求的情况下使用。定义long类型的值时在数字后加L: long aLong = 1234L。Java8之后，长整形也支持无符号位64位，$0$-$2^(64)-1$。长整形的包装类型为Long，初始值为0L。

### float,Float
float是单精度32位浮点数，值得范围还有待讨论[Floating-Point Types, Formats, and Values](https://docs.oracle.com/javase/specs/jls/se7/html/jls-4.html#jls-4.2.3)，需要注意，浮点数不是精确的值，所以在有需要精确控制的地方一定要注意(货币，价钱方面)，可以使用BigDecimal来代替浮点数运算。定义float的时候一定要加f，不然系统会默认为double。比如 float aFloat = 0.1f。flocat的包装类型为Float，初始值为0.0f。

### double,Double
double是双精度64位浮点数，遵从IEEE754。double也不是精确的，所以和float需要注意点基本类似。double的包装类型为Double，初始值为0.0d。

### boolean,Boolean
boolean数据类型仅仅有两个可能值，true，false。它在某些情况下占用一位，但是它的占用大小是一个不确定的。boolean的包装类型为Boolean，初始值为false。

### char
char 的数据类型是一个16位的Unicode字符类型，最小值\u0000(0)-最大值\uffff(65535)，初始值为\u0000


>可以总结一下，基本数据类型的包装类型都是首先字母大写，包装类型的主要作用是面向对象，增加更多的可操作性。基本数据类型作形参的时候是值传递，不会改变原来的值。特别注意String不是基本数据类型，只是String的对象都存储在静态区，是一个不可变对象,String 默认值为“null”。
>void 和 Void 也需要注意，在博客上有人把他们归类为第九大数据类型，但是官网却没有说明。参考[Primitive Data Types](http://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)

