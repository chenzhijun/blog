---
title: 基础排序算法
copyright: true
date: 2019-09-10 21:35:29
tags: 排序算法
categories: 数据结构与算法
---


# 基础排序算法

每次一看算法就是，嗯，看懂了，每次自己写代码就是，嗯？这怎么写？？ 本文将自己理解的冒泡排序，插入排序，选择排序三种做一个总结。


<!--more-->
## 冒泡排序

冒泡排序其实很形象，就是每次选最大，或者最小的值，与第一个值交换，逐次冒泡。就像水里面的泡泡，轻的泡泡总会浮起来一样。

实现方式：

```java
    int[] a = new int[]{1, 7, 5, 9, 12, 3, 5};

    public static int[] bubblingSort(int[] a) {
        for (int i = 0; i < a.length; i++) {
            for (int j = i; j < a.length; j++) {
                if (a[i] < a[j]) {
                    int tmp = a[i];
                    a[i] = a[j];
                    a[j] = tmp;
                }
            }
            System.out.println("第"+i+"次:");
            for (int an : a) {
                System.err.print(an + "\t");
            }
            System.out.println();
        }
        return a;
    }

```

假设有原始数组a,每次冒泡的结果就如下，我们这里选的是从大到小冒泡：

```shell

第0次:
12	1	5	7	9	3	5	
第1次:
12	9	1	5	7	3	5	
第2次:
12	9	7	1	5	3	5	
第3次:
12	9	7	5	1	3	5	
第4次:
12	9	7	5	5	1	3	
第5次:
12	9	7	5	5	3	1	
第6次:
12	9	7	5	5	3	1

```

冒泡排序的时间复杂度为O(n^2) ; 空间复杂度为O(1);是一个稳定算法。

> 稳定的意义是指，两个相同的数据，它们排序完之后相对位置不变，比如上面数组的5[a],5[b],排序完之后不会出现5[b],5[a]; a,b为他们的相对位置。

## 插入排序

插入排序的意思就是有一个数组，你可以假定左边的部分是有序的，这个时候你从右边无序的数据中，找出一个，然后将其插入到有序数据的数组中。
有点类似打扑克，如果一次性将牌发完，你总得给牌排个序吧，比如从左到右，左边第一张你可以假设这一张是有序的，从第二张开始，你就比较一下第二张和第一张谁大，小的就往前面移动，大的就往后面移动。如果是第三张，依次与第二张比，比完再与第一张比。所以第n张就是与n-1比，再与n-2,n-3，...比较，一直比到有一张比它小的，那么这个时候，就位置就对了。看起来就像是我们将第n张牌，插入到了之前有序的一个数组中。

算法实现：

```java
    public static void main(String[] args) {
        int[] a = new int[]{1, 7, 5, 9, 12, 3, 5};
        int[] b = insertSort(a);
        for (int i : b) {
            System.out.print(i + "\t");
        }
    }

    public static int[] insertSort(int[] arr) {

        if (arr.length == 1) {
            return arr;
        }
        for (int i = 1; i < arr.length; i++) {
            System.out.println("第" + i + "次," + "arr[i]为:" + arr[i] + " :");
            printArr(arr);
            for (int j = i; j > 0; j--) {
                if (arr[j] < arr[j - 1]) {
                    int tmp = arr[j];
                    arr[j] = arr[j - 1];
                    arr[j - 1] = tmp;
                }
            }
            printArr(arr);
        }

        return arr;
    }

```

执行结果：

```shell

 第1次,arr[i]为:7 :
 1	7	5	9	12	3	5
 1	7	5	9	12	3	5
 第2次,arr[i]为:5 :
 1	7	5	9	12	3	5
 1	5	7	9	12	3	5
 第3次,arr[i]为:9 :
 1	5	7	9	12	3	5
 1	5	7	9	12	3	5
 第4次,arr[i]为:12 :
 1	5	7	9	12	3	5
 1	5	7	9	12	3	5
 第5次,arr[i]为:3 :
 1	5	7	9	12	3	5
 1	3	5	7	9	12	5
 第6次,arr[i]为:5 :
 1	3	5	7	9	12	5
 1	3	5	5	7	9	12
 
```

插入牌的时间复杂度为：O(n^2), 空间复杂度为O(1), 是一个稳定排序算法。


## 选择排序

选择排序其实就是你拿了一手牌，每次你扫描一遍，拿到最小的那张，把它跟第一张交换下位置。第一次交换第一张位置的，第二次交换第二张位置的，之后依次交换到最后一张。这个咋一看挺像冒泡的，但是远离不相同，冒泡是每次都会有顺移的操作，比如`2,3,4,5,1`；如果你选了1，它要跟5做比较，交换：`2,3,4,1,5`；跟4做比较，交换`2,3,1,4,5`。而选择排序则是：`2,3,4,5,1`；你先选最小的1，然后跟第一个位置的2做交换变成了`1,3,4,5,2`,有没有发现，其实只做了一次交换。

代码实现：

```java
    public static void main(String[] args) {
        int[] a = new int[]{1, 7, 5, 9, 12, 3, 5};
        int[] b = selectSort(a);
        for (int i : b) {
            System.out.print(i + "\t");
        }
    }
    public static int[] selectSort(int[] arr) {
        for (int i = 0; i < arr.length; i++) {
            int j = i;//初始化j的位置
            int k = j;//数组中值最小的数的位置
            int tmp = arr[i];//用来存储每次遍历的最小值
            for (; j < arr.length; j++) {//遍历数组从i开始，0-i之间的数据可以当做是已经选择了最小的有序数组
                if (arr[j] < tmp) {
                    tmp = arr[j];
                    k = j;
                }
            }
            arr[k] = arr[i];//交换当前i的位置和最小值的位置k两个数据
            arr[i] = tmp;
        }
        return arr;
    }
```

选择排序算法的时间复杂度为：O(n^2);空间复杂度为：O(1); 稳定性为：不稳定。
