---
title: 排序算法-快速排序
date: 2017-03-16 23:27:29
tags:
	- 算法
categories: 算法
---

## 快速排序

### 介绍
快速排序，听名字就能感觉到主要讲究的就是快速。它其实主要采用分治法，选取基准元，然后把小的放左边，大的放右边，遍历一遍之后就能确定基准元的位置。之后以确定后的基准元位置做左右分割，就是分治了。
不说啥，直接上图：
![快速排序动态图-wiki](/images/Sorting_quicksort.gif)
<!--more-->

### 代码实现
```java
/**
 * Created by chenzhijun on 3/16/17.
 */
public class QuickSort {

    public static void sort(int[] a, int i, int j) {

        if (i > j) {   // 递归程序要有终止条件
            return;
        }

        int basic = a[i];  //设置基准元

        int start = i;  // 左右位置
        int end = j;

        while (start != end) {
            while (a[end] >= basic && end > start) { // 必须先从右边先开始，即基数对面
                end--;
            }
            while (basic >= a[start] && start < end) {
                start++;
            }

            if (start < end) {   //交换位置
                int temp = a[start];
                a[start] = a[end];
                a[end] = temp;
            }
        }

        a[i] = a[start]; // 确定基准元的位置
        a[start] = basic;
        sort(a, i, start - 1);  // 分割基准元左右两边，找出下一个排序组。   java 非基本类型传递，都是传递的引用传递。地址。
        sort(a, end + 1, j);
    }

    public static void main(String[] args) {
        int[] a = new int[]{1, 8, 2, 6, 5, 7, 2, 9};
        sort(a, 0, a.length - 1);
        for (int i : a) {
            System.out.print(i + ",");
        }
    }

}

```
![啊哈算法的一幅图](/images/quick-sort.png)

我们在程序的中设置的哨兵就是start和end，这里一定要注意，如果你的基准元是选取的第一个位置或者最后一个位置，那么循坏时候一定的是对面开始。

### 算法分析
快速排序的空间占用是$O(n)$,最坏情况下时间复杂度为$O(n^2)$,平均情况下是$O(nlogn)$。最坏情况下是怎么出来的呢？当每次分完之后一边是1个元素，一边是n-1个元素，这种情况下，时间复杂度就是最坏的了。至于$O(nlogn)$,这个可以计算出来的，也可以看这篇链接：[快速排序算法的时间复杂度分析](http://www.cnblogs.com/pugang/archive/2012/07/02/2573075.html)
