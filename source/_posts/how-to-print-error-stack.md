---
title: 异常堆栈信息打印
date: 2017-11-02 11:06:38
tags:
	- Java
categories: Java
---

## 异常堆栈信息打印

最近在开发积分和优惠券，忙的焦头烂额，基本上每天都在想着代码怎么写，功能怎么实现。整个架构是怎样。没有产品经理来梳理需求，基本上靠自己写。唉~ 。比较痛苦的是，我开发完功能，然后交互自己又给我随意乱加功能，导致我有些代码的重写，当然直接接口的封装而已，但是这种感觉很不爽，我都开发完了，你也不知会我一下，就乱来。
<!--more-->
不扯了。今天要记录下一个功能，就是将异常信息，通过邮件发送给我。过程中遇到一个问题，怎样获取到异常堆栈信息并将它输出为字符串，来通过邮件的形式发送出来。

其中最为主要的就是获取异常堆栈，并将之输出为字符串；

下面是一个实例：

```java
    @org.junit.Test
    public void testNull() {
        try {
            testNullException();
        } catch (Exception e) {
            StringWriter sw = new StringWriter();
            PrintWriter pw = new PrintWriter(sw);
            System.out.println("e.getCause() : " + e.getCause());
            System.out.println("e.getSuppressed() ： " + e.getSuppressed());
            System.out.println("e.getStackTrace() ： " + e.getStackTrace());
            System.out.println("e.getLocalizedMessage() ： " + e.getLocalizedMessage());
            System.out.println("e.getMessage() ： " + e.getMessage());
            System.out.println("e.getClass() ： " + e.getClass());
            /**
             * e.getCause() : null
             e.getSuppressed() ： [Ljava.lang.Throwable;@5b464ce8
             e.getStackTrace() ： [Ljava.lang.StackTraceElement;@57829d67
             e.getLocalizedMessage() ： null
             e.getMessage() ： null
             e.getClass() ： class java.lang.NullPointerException
             */
            System.out.println("==================");
            e.printStackTrace(pw);
            System.out.println("e " + sw.toString());
        }
    }
    private void testNullException() {
        List<String> list = null;
        System.out.println(list.get(0).toString());
    }
```

其中的主要原理就是通过`throwable`的`printStackTrace(pw)`将输出输出到流中，然后通过字符流将其中的数据转换成字符。所以我们可以进行一些封装：

```java
    public static String getStackTrace(Throwable throwable) {
        StringWriter sw = new StringWriter();
        PrintWriter printWriter = new PrintWriter(sw);

        try {
            throwable.printStackTrace(printWriter);
            return sw.toString();
        } finally {
            printWriter.close();
        }
    }
```

可能你会想说，为啥`StringWriter`没有在`finally`关闭，其实不是不关闭，而是`sw.close()`它就是一个空实现，调用它的`close`方法，其实也没做操作。

有问题欢迎留言交流，或者联系。