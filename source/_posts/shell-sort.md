---
title: 希尔排序算法
date: 2018-03-17 10:46:13
tags: 算法
categories: 算法
---

# 希尔排序算法

这个算法是插入排序的基础上做的优化，它描叙与实现可以看这个[插入排序](http://chenzhijun.top/2018/03/09/%E5%88%9D%E7%BA%A7%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/#%E6%8F%92%E5%85%A5%E6%8E%92%E5%BA%8F)。

希尔排序的思想是使间隔为h之间的元素都是有序的。
比如我们说有10个数字，首先我们使用间隔4，那么就是位置为 `1，5`,`2，6`,`3，7,`4，8`,`5，9`；将这些组的相应**位置的值**，我们说的1-9是指的位置，而不是值哦。将这些组进行排序。之后我们再进行第二次分组，也就是将间隔的长度再缩小，假如这里缩小为2，那么第二次的间隔位置组就是：`1，3，5，7，9`,`2，4，6，8`。这样循环比较进行交换之后，我们最终就可以将间隔降为1，这样我们就可以得到一个优化的优化后的插入排序了，也就是常说的希尔排序；

一般来说我们通常将长度设为2的幂除。也就是先用数组长度除2，再除2，再除2；
```java

public static void (int[] arr){
    for(int h=arr.length/2;h>0;h=h/2){
        for(int i=gap;i<arr.length;i++){
            for(int j=i;j<arr.length&&arr[j]<arr[j-h];j-=h){
                swap(arr,j,j-h)
            }
        }
    }
}

```


在<<算法>>第四版中看到另一种实现方式：

```java

public static void sort(int[] arr){
    int len = arr.length;
    int h = 1;
    while(h<arr.length/3){
        h=h*3+1;
    }

    while(h>=1){
        for(int i=h;i<len;i++){
            for(int j=i;j<len&&a[j]<a[j-h];j-=h){
                swap(arr,j,j-h);
            }
        }
        h=h/3;
    }
}

```

```java
public static void swap(int[] arr,int j,int h){
    arr[j] = arr[j]+arr[i];
    arr[i] = arr[j]-arr[i];
    arr[j] = arr[j]-arr[i];
}
```