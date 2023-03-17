---
title: 自己用 Java 写一个阻塞队列(二)
copyright: true
date: 2021-06-03 21:39:13
tags: Java
categories: Java
---

# 自己用 Java 写一个阻塞队列(二)

上一章 [自己用 Java 写一个阻塞队列](http://chenzhijun.top/2020/05/27/java-thread-lock-condition-blockqueue/) 我们写了一个初始阻塞队列。过了几天再回头看。嗯，好像看出哪里不对劲了。尝试改了一下。

## 原先队列实现的问题

我们说是队列，但是队列的特点是什么？先进先出！在上一个版本中，我们在出队列的时候用的是`size`值，也就是最后一个位置值。那么想想，我们这算队列么？这是栈啊。这是问题一。
另外我们知道 arraylist 其实其底层实现还是数组，那为啥我不直接用数组了？我基本上也不用 list 的功能啊。那这个地方就是第二个可以优化的地方了。
在第一个版本中为了运行正常，我把 size 的初始值定为 -1，我有点受不了，不行，这个地方我也要优化。
嗯，就优化这三个地方吧。

## 优化问题 ArrayList

其实际很简单，把 ArrayList 改为数组 Object[]。原先 add 和 get 的地方直接使用 Object[idx] 代替即可。

```java
Object[] objects = new Object[10];
objects[size] = t;
t = objects.[size];
```

## 优化入队和出队

队列的本质应该是先进先出的数据结构。这里用两个 idx 来代替，一个是入队索引，一个是出队索引。

```java
int enqIdx = 0;//入队的索引号
int deqIdx = 0;//出队的索引号
```
这里有个问题，就是当索引到了我们的数组大小的时候，要把索引重新开始计数。

```java
if((enqIdx+1)==objects.length){
    enqIdx=0;
}
if((deqIdx+1)==objects.length){
    deqIdx=0;
}
```

## 开始代码

<!--more-->
```java

import java.util.Random;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Item15_Lock_Condition {

    public static void main(String[] args) throws InterruptedException {

        BlockedQueue blockedQueue = new BlockedQueue();


        int i = 0, j = 0;
        while (i++ < 30) {
            Thread thread = new Thread("producer-" + i) {
                @Override
                public void run() {
                    while (true) {
                        System.out.println(this.getName() + ",生产数字:" + blockedQueue.enq(new Random().nextInt(10)));
                        try {
                            Thread.sleep(1000 * new Random().nextInt(10));
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            };
            thread.start();
        }

        while (j++ < 30) {
            Thread thread1 = new Thread("consumer-" + j) {
                @Override
                public void run() {
                    while (true) {
                        System.out.println(this.getName() + ",消费数字:" + blockedQueue.deq());
                        try {
                            Thread.sleep(1000 * new Random().nextInt(10));
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            };
            thread1.start();
        }
        while (true) {
            Thread.sleep(1000 * 60 * 60);
        }
    }

}

class BlockedQueue {
    final Lock lock = new ReentrantLock();
    Object[] objects = new Object[10];
    int size = 0;//容量值，从 0 开始计算。
    int enqIdx = 0;//入队的索引号
    int deqIdx = 0;//出队的索引号
    //判断是否满了
    Condition notFull = lock.newCondition();
    //判断是否空的
    Condition notEmpty = lock.newCondition();


    //入队
    Object enq(Object t) {
        lock.lock();
        try {
            //判断队列是否已满
            while (size == objects.length) {
                System.out.println("不好意思，满了！！！");
                notFull.await();
            }
            objects[deqIdx] = t;
            //如果当前入队是最后一个位置，那我们下次就要从 0 开始计算
            if ((deqIdx + 1) == objects.length) {
                deqIdx = 0;
            }
            size++;
            notEmpty.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
        return t;
    }

    //出队
    Object deq() {
        lock.lock();

        Object t = null;
        try {
            while (size == 0) {
                System.out.println("不好意思，空了~~");
                notEmpty.await();
            }
            t = objects[deqIdx];
            //如果当前已经是最后的一个索引并且已经出队了，那我们下次就要从 0 开始计算
            if ((deqIdx + 1) == objects.length) {
                deqIdx = 0;
            }
            size--;
            notFull.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
        return t;
    }

}
```

嗯，看起来有点样子了。不再像之前的感觉了。 毕竟这个版本是正经的队列的先进先出了。而且看不惯的从 -1 计数也改了。其实有时候多加几个指针索引也挺好的。