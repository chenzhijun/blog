---
title: ThreadLocal 类解析
date: 2017-09-28 17:27:01
tags:
    - Java
Categories: Java
---

## ThreadLocal 类解析

### 什么是ThreadLocal

JDK里面关于`ThreadLocal`的解释就是：它提供一个线程局部变量，很拗口对不对，没错就是这么拗口，所以你用的少，我们都用的少。它其实就是说，在一个类里面，这个类可以被很多线程访问，那么`ThreadeLocal`给这些线程，每一个分配一个他们自己私有的局部变量。还是拗口对不对？不着急，现在只要知道，它就是给多线程环境下，每个线程分配一个单独的变量。

### ThreadLocal 的组成

我们用idea看看`ThreadLocal`里面含有什么东西：
![2017-09-29-17-21-04](/images/qiniu/2017-09-29-17-21-04.png)

初一看貌似挺多的。其实在这个里面它主要实现了一个Map，如果你还记得它是为每个线程分配一个独立的变量，那么也就不难理解，它其实就是往map里面给每个线程设置了一个变量给他们使用。
然后看看`ThreadLocal`，主要就是几个方法：`get()`,`set(T)`,`initialValue()`,`remove()`，其中`initialValue`是`protected`:
<!--more-->
```java
    protected T initialValue() {
        return null;
    }
```
它主要是提醒所有的继承者，它需要初始化值。

然后看看`get()`:
```java
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```
从这里就可以看到一个事情：get的时候它用到了`Thread.currentThread()`，然后又用`getMap(threade)`。为啥会这样？它貌似使用的`thread`做的键，对么？也就是当前线程。没错，一个`map`里面它用当前线程做`key`，然后取到当前*线程*的value，而不会影响到其它的线程。顺便看下`ThreadLocalMap`类,看到它是一个`static class ThreadLocalMap {}`,这是不是就是静态变量唯一了？

其它的set(),remove()就不多说了，肯定也是操作这个map。

### ThreadLocal 使用

讲了一些简单的源码，貌似也没弄明白到底怎么用它，使用它又有什么效果？我们接下来看下实例，我们使用一个序列号生成器来看下实际效果。
首先我们定义一个序列号接口，主要用来获取序列号`Sequence.java`：

```java
/**
 * @author chen
 * @version V1.0
 * @date 2017/9/28
 */
public interface Sequence {
    int getNumber();
}
```

然后我们写一个生成序列号任务类`ClientThread.java`：

```java
/**
 * @author chen
 * @version V1.0
 * @date 2017/9/28
 */
public class ClientThread extends Thread {
    private Sequence sequence;

    public ClientThread(Sequence sequence) {
        this.sequence = sequence;
    }

    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            System.out.println(Thread.currentThread().getName() + " => " + sequence.getNumber());
        }
    }
}
```

现在在客户端A我们开启三个任务，首先不用`ThreadLocal`:

```java
/**
 * @author chen
 * @version V1.0
 * @date 2017/9/28
 */
public class SequenceA implements Sequence {
    private static int number = 0;


    public int getNumber() {
        number = number + 1;
        return number;
    }

    public static void main(String[] args) {
        Sequence sequence = new SequenceA();

        ClientThread t1 = new ClientThread(sequence);
        ClientThread t2 = new ClientThread(sequence);
        ClientThread t3 = new ClientThread(sequence);

        t1.start();
        t2.start();
        t3.start();
    }
}
```

控制台结果为：
![2017-09-29-17-43-38](/images/qiniu/2017-09-29-17-43-38.png)
可以看到每个线程对于number都是直接加的，这和static变量有关，但是我们如果想要每个线程都有单独的属性`number`了？只对我当前线程可以用，别人都不可以用了？
我们加上ThreadLocal看看：

```java

/**
 * @author chen
 * @version V1.0
 * @date 2017/9/28
 */
public class SequenceB implements Sequence {

    private static ThreadLocal<Integer> number = new ThreadLocal<Integer>() {
        @Override
        protected Integer initialValue() {
            return 0;
        }
    };


    public int getNumber() {
        number.set(number.get() + 1);
        return number.get();
    }

    public static void main(String[] args) {
        Sequence sequence = new SequenceB();

        ClientThread t1 = new ClientThread(sequence);
        ClientThread t2 = new ClientThread(sequence);
        ClientThread t3 = new ClientThread(sequence);

        t1.start();
        t2.start();
        t3.start();
    }
}

```

在看下结果:
![2017-09-29-17-46-34](/images/qiniu/2017-09-29-17-46-34.png)

可以看到`number`的值在每个线程里面都是单独私有的，线程0不会影响线程1的`number`值。没错，这就是ThreadLocal了。


参考资料：
[ThreadLocal 那点事](https://my.oschina.net/xianggao/blog/77232)
[ThreadLocal 那点事-黄勇](http://my.oschina.net/huangyong/blog/159489)
[ThreadLocal 那点事(续集)](https://my.oschina.net/huangyong/blog/159725)