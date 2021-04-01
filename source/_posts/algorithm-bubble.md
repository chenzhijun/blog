---
title: 排序算法-冒泡排序
date: 2017-03-16 12:41:36
tags:
	- 算法
categories: 算法
---

## 冒泡排序

### 排序原理

冒泡排序是将最大或者最小的值先挑出来，然后依次类推，最终使得整个数组有序。比如我们在学校操场排队的时候，老师先叫你站好，站好之后，根据你们的身高找到一个最矮的，排到第一个，然后是第二个，然后这样依次类推...到最后就是一个身高顺序的队伍了。

![](/images/bubblesort.png)
<!-- more -->
### 代码实现


```java

public class Main {

    public static void main(String[] args) {
        int[] a = new int[]{12, 35, 99, 18, 76};

        for (int i = 0; i < a.length; i++) {
            for (int j = 0; j < a.length - i - 1; j++) { // 如果已经冒泡出来了i个数，则已经有i个数排好序，就无需再比较了 
                if (a[j] < a[j + 1]) {//相邻两个位置交换
                    int t = a[j];
                    a[j] = a[j + 1];
                    a[j + 1] = t;
                }
            }
        }

        for (int i : a) {
            System.out.print(i + ",");
        }
    }

}

```
这个是冒泡排序的代码，第一个for循环，指的是有多少个数需要进行排序；第二个for循环开始交换位置，将小(大)的数一个一个交换到结尾的位置。

```java

public class Main {

    public static void main(String[] args) {
        int[] a = new int[]{1, 5, 4, 5, 2, 32, 2};

        for (int i = 0; i < a.length; i++) { // 先确定位置
            for (int j = 0; j < a.length; j++) { //确定位置的大小
                if (a[i] < a[j]) { // 找到第i个位置，然后再第i个位置找到最优值
                    int t = a[i];
                    a[i] = a[j];
                    a[j] = t;
                }
            }
        }

        for (int i : a) {
            System.out.print(i + ",");
        }
    }
}

```
这个代码是我自己写的，也是可以排序，只是这里将a[i]也用作交换基数（上面代码，a[i]没有参与运算），我这个算法貌似也算冒泡吧？

### 算法分析
冒泡代码实现里面可以看到有两个for循环，时间复杂度为$O(N^2)$，空间复杂度为N。这个意思是什么？假设计算机每秒运行1亿次，对1亿个数排序，冒泡排序需要1亿秒(现代计算机肯定比这个快)。一天是多少秒了？86400秒。排序时间约等于1150天~，因此冒泡排序虽然易懂，很形象，但是一般公司都不会用到它。


