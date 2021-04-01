---
title: Java HashMap
copyright: true
date: 2020-03-25 22:57:59
tags: Java
categories: Java
---

# HashMap 介绍

这个该怎么写了？其实网上好多博客说了这个，其实我看了之后，甚至看了源码之后我也不知道怎么写，源码中Hashmap的介绍大致如下：

```
Hashmap 大致是等于hashtable的，除了非同步和允许null值之外；
不保证数据的顺序；
常数级别的get和put；
最重要性能的两个属性： capacity  ， loadfactor
0.75的加载因子数是一种空间和时间上的tradeoff（平衡）
相同的hashcode()会降低它的性能
hashmap的实现是非同步的；
iterator fail-fast ConcurrentModificationException.
```
首先我们介绍下它的内部数据结构吧。

## HashMap的内部结构
<!--more-->
Hashmap 是由`Node<K,V>[] table`和链表组成的结构，数组被分为一个个的桶（bucket），通过 hash 值在这个桶上进行寻址。hash 值相同的键值对就以链表的形式存储，如果链表的大小超过 `TREEIFY_THRESHOLD=8`就会将链表进行树化。

![2020-03-26-22-27-21](/images/qiniu/2020-03-26-22-27-21.png)

接下来我们看看它的一个初始化方法：

```java

    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
```

在 new 一个 hashmap 的时候会发现，`Node<K,V>[] table;` table 就没有设值。

然后我么来看一个 put 方法：

```java
 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 开始进行resize()保证桶的容量； 
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //i = (n - 1) & hash 是不是最后一个桶，并且为null
        if ((p = tab[i = (n - 1) & hash]) == null)
        //直接加到tab后面
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            //
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 进行树化
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
总的一个put流程大致就是： 初始化或者 resize() 扩容和树化；

现在我们看看 resize() 方法，

```java
 final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;//threshold=capacity * load factor
        int newCap, newThr = 0;
        if (oldCap > 0) {
            //是否超过最大值 MAXIMUM_CAPACITY = 1 << 30; 2 的 30 次方
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //两倍扩容
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
        // 桶的容量默认值： DEFAULT_INITIAL_CAPACITY = 1 << 4； 默认值 16；
            newCap = DEFAULT_INITIAL_CAPACITY;
        // 桶扩容的阀值：
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        //设置扩容的阀值
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        //如果原来的桶中不为null，扩容后要把桶中的数据迁移到新桶中。
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                       // 会讲原来的一个链条打散成两条链
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            //这里也就是会在高并发下导致hashmap出现问题的原因
                            next = e.next;
                            //这里的 e.hash & oldCap ==0 可以有效避免重复计算hash值，而且把原来的桶中重复的值分散到新的桶中。
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

看完 resize 就会发现发现它做了很多事，初始化，扩容，数据复制；数据复制也会造成一定的开销。
另外resize中的防止rehash并且分散之前的冲突节点的算法也很巧妙：
![2020-03-27-23-13-43](/images/qiniu/2020-03-27-23-13-43.png)

另外我们看下树化：`treeifyBin(tab, hash);`

```java
 final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        //如果是空桶或者桶里面的数据少于 MIN_TREEIFY_CAPACITY = 64;只是简单扩容就行了。
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```

所以可以看到 HashMap 的 `LOAD_FACTOR` 加载因子和容量是多么的重要了。我们一般的化可以优化为：
`负载因子 * 容量 > 元素数量` 即初始化容量时候，值要大于“预估元素数量 / 负载因子” 

hashmap在高并发场景下会发生什么了？
要记住，hashmap本身就被声明为了非同步安全的类，如果在多线程环境下是会有可能导致无限循环占用cpu，size不准确，具体的原因可以看下这篇博客 [a beatiful race condition](https://mailinator.blogspot.com/2009/06/beautiful-race-condition.html)

那么问一下，为什么要进行树化了？
本质上是个安全问题，如果一个对象hash冲突，都被放到一个桶里面，就会形成一个链表，链表的查询是线性的，就会严重影响存储的性能；
另外有种安全攻击叫做“哈希碰撞拒绝服务攻击”，就是构建哈希冲突的数据，恶意代码用这些数据与服务器进行交互，导致服务端cpu大量占用。来达到攻击的目录。

那么，并发下，我们应该怎么使用HashMap了？
明天讲～

<!--

但是假如你在面试，面试官会怎么问你了？

咳咳，模拟下：

Q：你好，请问你们平常用的jdk版本是多少？
A：线上主要用的是jdk8
Q：那你能介绍下常见的这个HashMap吗？
A：balabla 上面一大段
Q：1.8中对hashmap有什么优化了？
A：在数据存储中引入了树化，在数据超过8的时候就会变成红黑树。
Q：为什么要引入树来做存储了？有什么好处了？
Q: 能不能介绍下hashmap put的整个过程？
Q: 能不能介绍下hashmap的使用场景？
Q：高并发的场景下应该怎么办了？ 





## 这个技术出现的背景、初衷和要达到什么样的目标或是要解决什么样的问题？

## 这个技术的优势和劣势分别是什么，或者说，这个技术的 trade-off （要什么不要什么）是什么

## 这个技术适用的场景（技术场景或业务场景）

## 这个技术的组成部分和关键点（核心思想，核心组件）

## 这个技术的底层原理和关键实现

## 已有的实现和它之间的对比
https://bugs.java.com/bugdatabase/view_bug.do?bug_id=6423457

-->


参考：https://www.cnblogs.com/Michaelwjw/p/6411176.html
