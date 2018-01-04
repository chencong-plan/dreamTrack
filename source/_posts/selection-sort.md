---
title: 算法篇——选择排序SelectionSort
date: 2017-12-20 13:27:33
tags: 
  - SelectionSort
  - 线性结构
  - 选择排序
categories:
  - 算法学习
---

## 前言

### 介绍原理
> 排序算法当中最开始学习的是选择排序，这样一个线性结构的排序算法。原理其实很简单。`每一趟从剩余待排序的记录当中选出最小元素，放在已排序的记录最后（替换位置），直到全部记录排列完成` 

同时选择排序并不是一个稳定的排序算法，但是是一个简单直观的排序算法。时间复杂度O(n²)，有些时候可以拿代码复杂度来换时间复杂度，同时在底层汇编等速度较快情况下可以更加容易的实现选择排序。

## 实现过程（Java）

### 代码实现
下面是我用Java实现的选择排序算法。
```java
/**
 * @author : ChenCong
 * @date : Created in 13:12 2017/12/20
 */
public class SelectionSort {

    public static void main(String[] args) {

        int[] array = {9,8,3,6,7,4,31,2,1};
        sort(array, 9);
        // 打印输出排序号的数组
        System.out.println(Arrays.toString(array));
    }

    public static void sort(int[] array,int n){
        for (int i = 0; i < n; i++) {
            // 将每一次排序过程都打印出来，可以省略
            System.out.println(Arrays.toString(array));
            int minIndex = i;
            // 在区间[i n) 当中寻找最小的数字
            for (int j = i; j < n ; j++) {
                if (array[j] <array[minIndex] ){
                    minIndex = j;
                }
            }
            if (i != minIndex){
                int temp = array[i];
                array[i] = array[minIndex];
                array[minIndex] = temp;
            }
        }
    }
}

```

## 分析及运行结果

### 代码原理分析
从上面代码可以直观的分析出来，选择排序在对数组`[0,n]`进行排序过程当中，最重要的一个思想就是 `遍历整个序列n,找出区间[i,n)当中最小的数字`然后跟当前位置进行替换。在上面代码当中` int minIndex = i;` 遍历数组时候，当起始位置设置为最小值，然后在`[i,n]`之间寻找比这个位置`minIndex`更小的值，如果有，则替换，如果没有则当前位置不动，查找`[i+1,n]`区间。如代码片所示：
```java
// 在区间[i n) 当中寻找最小的数字
for (int j = i; j < n ; j++) {
    if (array[j] <array[minIndex] ){
        minIndex = j;
    }
}
```
替换两个索引位置的值，就是简单的替换。可以引入中间值`temp`也可以之间使用`a=a+b,b=a-b,a=a-b`的形式。
替换代码如下：
```java
 if (i != minIndex){
    int temp = array[i];
    array[i] = array[minIndex];
    array[minIndex] = temp;
}
```
或者
```java
if (i != minIndex) {
    // a= 2,b = 1;
    // a = a+b; 
    // b = a-b;
    // a = a-b;
    array[i] = array[i] + array[minIndex];
    array[minIndex] = array[i] - array[minIndex];
    array[i] = array[i] - array[minIndex];
}
```

### 运行结果
不管使用上面哪一种替换方法，运行得到的结果都是一样的，对数组`[9, 8, 3, 6, 7, 4, 31, 2, 1]`进行排序，上面提到了我将每一次排序的结果进行打印.
```java
[9, 8, 3, 6, 7, 4, 31, 2, 1]  // 原始数据 9为起始位置，剩余数据中寻找最小值1，与9的位置替换，得到的结果
[1, 8, 3, 6, 7, 4, 31, 2, 9]  // 8 为起始位置，在剩余数据中寻找到最小值2,与8位置替换，得到的结果
[1, 2, 3, 6, 7, 4, 31, 8, 9]  
[1, 2, 3, 6, 7, 4, 31, 8, 9]
[1, 2, 3, 4, 7, 6, 31, 8, 9]
[1, 2, 3, 4, 6, 7, 31, 8, 9]
[1, 2, 3, 4, 6, 7, 31, 8, 9]
[1, 2, 3, 4, 6, 7, 8, 31, 9]
[1, 2, 3, 4, 6, 7, 8, 9, 31]
[1, 2, 3, 4, 6, 7, 8, 9, 31]  // 遍历整个数组之后，得到的结果
```
起始位置`minIndex`遍历整个数组之后就可以完成`SelectionSort`选择排序。
将上面`sort(int[] array,int n)`方法还可以省略参数`n`，优化如下
```java
  /**
     * 对数组进行排序——选择排序
     *
     * @param array 将要排序的数组
     */
    public static void sort(int[] array) {
        for (int i = 0; i < array.length; i++) {
            // 将每一次排序过程都打印出来，可以省略
            System.out.println(Arrays.toString(array));
            int minIndex = i;
            // 在区间[i n) 当中寻找最小的数字
            for (int j = i; j < array.length; j++) {
                if (array[j] < array[minIndex]) {
                    minIndex = j;
                }
            }
            if (i != minIndex) {
                int temp = array[i];
                array[i] = array[minIndex];
                array[minIndex] = temp;
            }
        }
    }
```

## 后记

### 时间复杂度
根据上面的结果来计算，外层循环移动确定`minIndex`为`O(n)`，在每一循环内还需要寻找`[i,n)`之间的最小值也是`O(n-1)`，因此时间复杂度为`O(n(n-1)) ≈ O(n²)`。

## 联系

[聪聪](https://ccoder.cc/)的独立博客 ，一个喜欢技术，喜欢钻研的95后。如果你看到这篇文章，千里之外，我在等你联系。

- [Blog@ccoder's blog](https://ccoder.cc/)
- [CSDN@ccoder](http://blog.csdn.net/chencong3139)
- [Github@ccoder](https://github.com/chencong-plan)
- [Email@ccoder](mailto:admin@ccoder.top) *or* [Gmail@ccoder](mailto:chencong3139@gmail.com)

