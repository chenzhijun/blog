---
title: 数据结构-队列
date: 2017-03-23 23:42:23
tags:
	- Algorithm

categories: Algorithm
---


## 数据结构-队列

>程序的开发越来越发现，重要的其实是一些基础的东西。程序=数据结构+算法；这个等式真的越来越让人有感悟。

今天遇到一个问题，有个妹子告诉了你一串加密后的qq号，解密的规则是第一个数删除第二个数排最后，第三个数删除，第四个数排最后，依次类推到没有一个数可以删除，按照删除的数的顺序就是解密后的qq号码。请问，如果你对妹子有想法的情况下，你该如何解决这个问题了？
<!--more-->
这个问题的解决方式可以用到队列，队列是一个先进先出的数据结构。实现队列的方式有很多，在Java里面可以用数组，或者用list，这种线性结构存储。下面看Java代码实现


#### 代码实现
```java
import java.util.ArrayList;
import java.util.Arrays;

/**
 * 解密qq号
 * 首先将第1个数删除，紧接着将第2个数放到这串数的末尾，
 * 再将第3个数删除并将第4个数再放到这串数的末尾，
 * 再将第5个数删除……直到剩下最后一个数，将最后一个数也删除。
 * 按照刚才删除的顺序，把这些删除的数连在一起就是QQ号了
 * <p>
 * Created by alvin on 3/23/17.
 */
public class Queue {

    public static void main(String[] args) {
        Integer[] qqNumber = new Integer[]{6, 3, 1, 7, 5, 8, 9, 2, 4}; //6 1 5 94 7 2 8 3
        ArrayList<Integer> queue = new ArrayList<Integer>();
        for (Integer i : qqNumber) {
            queue.add(i);
        }

        int size = queue.size();

        System.out.println("加密后qq:" + Arrays.toString(queue.toArray()));
        String qq = "";
        for (int i = 0; i < size; i++) {
            if (i % 2 == 0) {
                qq += queue.get(i);
                continue;
            }
            queue.add(queue.get(i));
            size++;
        }

        System.out.println("解密后qq:" + qq);


    }


}
```

代码实现的不是很优雅，但是应该算是很明了的。ArrayList就是我们要用来做破译的队列。每经过一个数，确认是删除，还是往后面继续排队。在第一个位子设置标记，每往list中增加一个元素，size就会+1，但是标记会一直移动，所以移动到两者相等的时候，即达到终点，这个时候我们就能得到正确的qq号了。

>ps: ArrayList的源码中，实现的方式是一个空的数组{},默认大小为10,ArrayList中有一个ensureCapacity方法来保证容量得到保证。在这个机制下很多人都习惯不设默认值，其实如果你的数据量是均衡的，既你能大概平均预计到最大值的情况下，最好是初始化list的容量，ensureCapacity 这种机制是通过直接将原list拷贝到新list，容量变为(原大小+原大小*2)，然后让引用指向新的list，而之前的旧list就没有用只能等待gc回收。想想看如果你平凡的去这种扩容(想想都浪费啊，程序员都是很抠的，尤其是压榨性能方面~~~)