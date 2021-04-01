---
title: Java设计模式-单例模式
date: 2017-09-23 16:53:22
tags:
   - Java
   - 设计模式
categories: Java
---

## 设计模式-单例模式

### 简述

单例应该是最简单也是最常用的模式了。单例模式通常用在当一个系统只需要唯一一个类，比如线程池，日志打印等。单例模式的类就只有一个，听起来很简单，但是要写好单例却不容易。不信你自己先想想怎么保证系统里面只有一个单例？

_单例模式：确保一个类只有一个实例，并且提供一个全局访问点_

### 单例实践

要保证单例的第一步就是要保证类不能随意实例话，new 一个对象的代价是很简单的比如：`new Object()`。但如果不能new了？没错，我们第一步就是让构造方法私有话，不会对外开放`private ObjectConstractor()`;但是如果我们还是需要使用这个类怎么办了？一个最简单的单例模式就可以写出来了：
<!--more-->

```java
public class Singleton {
    private static Singleton instance;//使用static来确保唯一。但是static真的管用么？

    public static Singleton getInstance(){
        if(instance==null){//先判断是不是存在，不存在就实例一个。
            instance = new Singleton();
        }
        return instance;
    }
    private Singleton(){

    }
}
```

其实你可以想到，如果在多线程的环境下，我们上面的代码还是会有问题的，就是线程A和线程B同时同刻读取instance，然后两者都是读到null，然后就AB都会去实例话一次instance，这跟我们预想的不一致。

嗯我们可以做点改进，那就是`同步`：

```java
    public static synchronized Singleton getInstance2(){
        if(instance==null){
            instance = new Singleton();
        }
        return instance;
    }
```
没错在上列中我们是可以解决掉多线程的情况下，instance实例话两次的情况的，这种方式我们也叫做`懒汉式单例模式`。但是我们引入了`synchronized`,这又会造成一个新的问题，那就是我们，我们知道这种同步是会降低性能的，当然如果你的机器配置够好，那么对此就可以完全忽略，但是作为程序员，总有一种想要压榨性能的天生渴望～

让我们再来做些改进：
```java

public class Singleton {
    private static Singleton singleton = new Singleton();

    private Singleton(){

    }

    public static Singleton getInstance(){
        return singleton;
    }
}
```

这种改进的方式是在jvm加载该类的时候就已经初始话了，JVM保证任何线程访问这个类之前一定先创建了该类，这种方式也被形象的称为`饿汉式单例模式`。

## 单例模式实现方法

### 懒汉式单例模式：

```java
package single;

public class Singleton {
    private static Singleton instance;

    public static synchronized Singleton getInstance(){
        if(instance==null){
            instance = new Singleton();
        }
        return instance;
    }
    private Singleton(){

    }
}
```

### 饿汉式单例模式：

```java
public class Singleton {
    private static Singleton singleton = new Singleton();

    private Singleton(){

    }

    public static Singleton getInstance(){
        return singleton;
    }
}
```

### 双重加锁校验：

```java
package single.doubblechecking;

public class Singleton {
    private volatile static Singleton singleton;

    private Singleton(){}

    public static Singleton getSingleton(){
        if(singleton == null){
            synchronized (Singleton.class){
                if(singleton == null){
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

### 静态内部类：

```java
public class Singleton {
    private static class SingletonInstance {
        private static final Singleton sinleton = new Singleton();
    }

    private Singleton() {
    }

    public static Singleton getSingleton() {
        return SingletonInstance.sinleton;
    }
}
```

### 枚举单例模式:

```java
package single.enums;

public enum Singleton {
    INSTANCE;

    private Object object;
    private Singleton(){
        // init something..
        object = new Object();
    }

    public void doSomething(){
        //....
    }

    public Object getObject() {
        return object;
    }
}
```
枚举的方式特别有意思，所以我额外还写了一个测试类：

```java
package single;

import single.enums.Singleton;

public class Demo {
    public static void main(String[] args) {
        Singleton instance = Singleton.INSTANCE;
        instance.getObject();

        System.out.println(instance.getObject());
        System.out.println(instance.getObject());
        System.out.println(instance.getObject());
        System.out.println(instance.getObject());
    }
}
```

这四次输出都是一样的：

```java
java.lang.Object@7f31245a
java.lang.Object@7f31245a
java.lang.Object@7f31245a
java.lang.Object@7f31245a
```

用枚举实现单例是<<Effective java>>里面提倡的，但是实际中我看很少有人用，我们公司也有人直接用的是懒汉式方法上加`synchronized`，历史的原因就不追述了，但是自己写的代码一定要好好的检查。

> 什么是好的代码？可能就是编写以后不需要再重构的代码。因为它的性能已经是最完美了，顶多也就是加一些判断逻辑而已。