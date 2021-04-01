---
title: 基础排序算法
copyright: true
date: 2019-09-13 19:14:07
tags: 排序算法
categories: 数据结构与算法
---


# 基础排序算法2

## 归并排序

归并排序的意义是将利用递归的思想，将一个大问题解决为可重复求解的小问题，最后合并这些小问题得出结果。
归并是直接数组对半分，分成两个子数组。然后对两个子数组再进行排序求解。
<!--more-->
```java
package me.chenzhijun;

/**
 * 归并排序：对半分成两个子数组，然后子数组再向下分，之后合并数组。
 */
public class MergeSort {

    public static void main(String[] args) {
        System.out.println(3 / 2);
        System.out.println(1 / 2);
        int[] a = new int[]{1, 7, 5, 9, 12, 3, 5};
        mergeSort2(a, 0, a.length - 1);
//        mergeSort(a, 0, a.length - 1);
        for (int i : a) {
            System.out.print(i + "\t");
        }
    }

    public static void mergeSort2(int[] arr, int start, int end) {
        if (start >= end) {
            return;
        }
        int middle = start + (end - start) / 2;
        mergeSort2(arr, start, middle);
        mergeSort2(arr, middle + 1, end);
        merge2(arr, start, middle, end);

    }

    private static void merge2(int[] arr, int start, int middle, int end) {
        //用来存储的临时数组
        int[] newArr = new int[end - start + 1];
        //p q 两个指针，遍历arr
        int p = start;
        int q = middle + 1;
        int i = 0;
        //[0 , middle] 和 [middle+1 , end] 的数据分别赋值给新数组
        while (p <= middle && q <= end) {
            if (arr[p] <= arr[q]) {
                newArr[i++] = arr[p++];
            } else {
                newArr[i++] = arr[q++];
            }
        }

        //判断两边谁还有剩余的数
        //1 假设左边有剩余的数
        int p1 = p;
        int q1 = middle;
        //2 判断是不是左边真的有剩余，下面的if可以判断出右边是不是有剩余，while中只会有一边清空时才会退出while
        if (q <= end) {
            p1 = q;
            q1 = end;
        }

        //将剩余的数填满新数组
        while (p1 <= q1) {
            newArr[i++] = arr[p1++];
        }

        //将新数组（已排序）的值替换老数组的位置
        //这里要注意start-end之间的个数其实就是newArr数组的个数为end-start , 所以这里只能让idx最大为end-start 也相当于为循环end-start次数
        for (int idx = 0; idx <= end - start; idx++) {
            arr[idx + start] = newArr[idx];
        }
    }


    public static void mergeSort(int[] arr, int start, int end) {
        if (start >= end) {
            return;
        }
        int q = start + (end - start) / 2;
        mergeSort(arr, start, q);
        mergeSort(arr, q + 1, end);
        merge(arr, start, q, end);
        return;
    }

    private static void merge(int[] arr, int start, int middle, int end) {
        int[] tmps = new int[end - start + 1];
        int i = 0;
        int p = start;
        int q = middle + 1;
        while (p <= middle && q <= end) {
            if (arr[p] <= arr[q]) {
                tmps[i] = arr[p];
                p++;
            } else {
                tmps[i] = arr[q];
                q++;
            }
            i++;
        }

        int i1 = p;
        int j1 = middle;
        if (q <= end) {
            i1 = q;
            j1 = end;
        }
        while (i1 <= j1) {
            tmps[i] = arr[i1];
            i1++;
            i++;
        }

        for (int j = 0; j <= end - start; ++j) {
            arr[start + j] = tmps[j];
        }

    }
}

```

## 快速排序

也是归并排序的一种，不过快排采用的是先分区，再归并。快排采用的是找一个基准点，然后比较数组中数据的基准点找出分割点，以分割点来割分子数组。

```java
package me.chenzhijun;

/**
 * 快排序的思想就是,同一个数组,不需要额外的空间,
 * 最重要的部分是找出每个数据的应该放置的位置，这个位置的左边全小于它，右边全大于它，
 * 它是一个分割点，然后依次类推，每个数据都放到一个这样的位置，数组就拍好顺序了。
 * 快排序：O(nlogn)
 * 空间：O(1)
 * 稳定性：不稳定，因为两两交换嘛。
 */
public class QuickSort {

    public static void main(String[] args) {
        System.out.println(3 / 2);
        System.out.println(1 / 2);
        int[] a = new int[]{1, 7, 5, 9, 12, 3, 5};
        quickSort(a, 0, a.length - 1);

        for (int i : a) {
            System.out.print(i + "\t");
        }
    }

    private static void quickSort(int[] arr, int start, int end) {

        if (start >= end) {
            return;
        }
        int n = quickSortN2(arr, start, end);
//        int n = quickSortN3(arr, start, end);
        quickSort(arr, start, n - 1);
        quickSort(arr, n + 1, end);
    }

    /**
     * 取末尾为参考值
     * @param arr
     * @param start
     * @param end
     * @return
     */
    private static int quickSortN2(int[] arr, int start, int end) {
        int pivot = arr[end];//要从尾部取这个参照点
        int p = start;//定一个指针

        for (int q = start; q <= end - 1; q++) {
            if (arr[q] < pivot) {//将p留在永远大于参考点的第一个位置
                swap(arr, p, q);
                p++;//顺着左移，保持第一个位置
            }
        }

        /**已经遍历完了p的位置，左边是小于参考点的数据，
         右边是大于参考点的数据，p的位置存放参考点，
         该位置就刚好变成了一个分割点。将一个大的分解了两个子问题，
         而子问题也可以重复刚刚的步骤，最后所有的数据都会待在那个最
         合适的位置，最后的顺序也就排列好了。
         */
        swap(arr, p, end);
        return p;
    }

    private static void swap(int[] arr, int p, int q) {
        int temp = arr[p];
        arr[p] = arr[q];
        arr[q] = temp;
    }

    /**
     * 选第一个为参考值。注意返回的p-1的值
     * 比选最后一个值为参考值多了几次+,-的操作
     * @param arr
     * @param start
     * @param end
     * @return
     */
    private static int quickSortN3(int[] arr, int start, int end) {
        int pivot = arr[start];
        int p = start + 1;

        for (int q = start + 1; q <= end; q++) {
            if (arr[q] < pivot) {
                int temp = arr[q];
                arr[q] = arr[p];
                arr[p] = temp;
                p++;
            }
        }
        //p的位置是大于参考点的,因此要交换p-1的位置
        swap(arr, start, p - 1);
        return p - 1;
    }

    /**
     * 第一版本（错误方法）
     * 有bug，当出现
     *
     * [8,1,9,12,6,0]  p=0;q=0
     * a: 1,8,9,12,6,0   p=1;q=1
     * b: 1,6,9,12,8,0   p=2;q=4
     * c: 1,6,0,12,8,9   p=3;q=5
     *
     * 这个时候返回p=3,最终的结果肯定不正确。
     *
     * @param arr
     * @param start
     * @param end
     * @return
     */
    private static int quickSortN(int[] arr, int start, int end) {
        int pivot = arr[start];
        int p = start;

        for (int q = start; q <= end; q++) {
            if (arr[q] < pivot) {
                int temp = arr[q];
                arr[q] = arr[p];
                arr[p] = temp;
                p++;
            }
        }
//        arr[p] = pivot;
        return p;
    }
}

```

## 计数排序

桶排序的一种特殊方式，数据必须是非负整数。桶排序的方式是将数据最小值和最大值均分为多个桶，把数据放到桶里面，然后依次取出桶中的数据。计数排序的实现：

```java
package me.chenzhijun;

public class CountingSort {

    public static void main(String[] args) {

        int[] a = new int[]{1, 7, 5, 9, 12, 3, 5};
        countSort(a);
        for (int i : a) {
            System.out.print(i + "\t");
        }
    }


    /**
     * 计数排序就是遍历原始数组A找到最大值，然后以（最大值+1）的长度建立一个新数组B
     * 遍历数组A,每遍历A的一个数据a，在B中a的位置就+1；B[a]=B[a]+1
     * 遍历数组B获取B[a]之前[0-a]的所有数据个数，目前A中已经遍历过一次了，A中的数
     * 据在B中对应的下标中都会有值，统计B中a之前的所有个数，这样我们就能知道A中的数
     * 据a在A数组中应该在哪个位置。B遍历完之后，就相当于知道了a和小于a的所有数据的
     * 个数之和。
     * 新建一个数组C,长度和A一致，我们遍历一下A，取出数据A[i],数据A[i]在C中的位置
     * 应该为在数组B[A[i]]的值，即小于或等于数据A[i]的个数，之后将C[B[A[i]]-1]赋值为A[i]；
     * B[A[i]]为个数
     * B[A[i]]-1为数组实际下边，数组从0开始
     *
     * @param arrA
     */
    public static void countSort(int[] arrA) {
        int max = arrA[0];
        for (int i = 0; i < arrA.length; i++) {
            if (arrA[i] > max) {
                max = arrA[i];
            }
        }

        int[] arrB = new int[max + 1];
        //第一步是找出相同值的个数，比如数组里面5的数据，有2个，那么newA中第5个位置为2
        for (int i = 0; i < arrA.length; i++) {
            arrB[arrA[i]]++;
        }

        //计算数组中当前位置的数据有几个，比如数组中5的数据，那么从0-5，总共有多少个数字了？
        // 这个for循环就是计算总共的数字。计算出数字了之后，我们就知道，5在数组中顺序的位置了
        for (int i = 1; i < arrB.length; i++) {
            arrB[i] = arrB[i] + arrB[i - 1];
        }

        //一个新数组，长度跟要排序的数组一样，当我们遍历一下要排序数组，取出里面的数据
        //根据取出的值，在newA中下表为值的位置取出个数,就是这个新数组的位置。
        int[] arrC = new int[arrA.length];
        for (int i = 0; i < arrC.length; i++) {
            int position = arrB[arrA[i]] - 1;//arrB 里面存储的是个数，在新数组就是位置，减1是因为从0计算嘛，所以减1
            arrC[position] = arrA[i];// position 位置要等于arr[i]，取出了一个值所以就
            arrB[arrA[i]]--;//取出一个数据了，个数就少一个了。
        }

        for (int i = 0; i < arrA.length; i++) {
            arrA[i] = arrC[i];
        }
    }
}
```

## 几种基础排序算法比较

![几种基础排序算法比较](/images/qiniu/2019-09-13-19-14-25.png)