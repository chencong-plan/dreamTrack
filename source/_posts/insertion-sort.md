---
title: 算法篇——插入排序InsertionSort
date: 2018-01-03 13:21:07
tags:
    - InsertionSort
    - 线性结构
    - 插入排序
categories:
    - 算法学习
---

## 前言

> `插入排序`就是每一步都将一个待排数据按照其大小插入到已排数据中合适位置，直至所有数据都排列完毕。我这里按照数据`从小到大`顺序排列。因此可以将上面原理简化成`每趟寻找array[i]这个元素的合适插入位置，将其放入其中` 直到所有数据(i=n)排列完毕。

## 插入排序介绍

### 介绍原理

插入排序是一种稳定的排序，上面提到了每一趟当中寻找`array[i]`的合适位置，如果此时它已经在合适位置就不需要进行移动了，可以省去一部分时间。内层循环进行的条件为`array[j] < array[j-1]` 。那么我们可以猜想如果`array[j-1]< array[j]` 此时元素在自己本该在的位置，就不需要进行一次循环。这种思想相对于选择排序不管元素是否在自己本应该在的位置上都需要进行寻找一遍，有了一定的优化。所以说插入排序在针对`几乎是有序`的序列进行排序的时候，效率会提高很多，最优化效率可以提高到`O(n)`.

### 代码实现

上面说了思想，下面就来看一看代码实现部分。

```java
/**
     * 插入排序算法,每一次寻找array[i]这个元素合适的插入位置。
     * 内层循环需要进行的条件就是array[j] < array[j-1] ，相对于选择排序这也是插入排序可以进行优化的地方。
     * 根据这个array[j]<array[j-1] 后面比前面大这个条件，就可以产生了第二种写法。
     *
     * @param array 将要进行排序的数组
     */
    public static void sort(Comparable[] array) {
        int n = array.length;
        for (int i = 1; i < n; i++) {
            // 寻找元素array[i]合适的插入位置
            // 当前位置和前一个位置进行比较 最多考察到j=1的情况
            // 写法一
            for (int j = i; j > 0; j--) {
                if (array[j].compareTo(array[j - 1]) < 0) {
                    swap(array, j, j - 1);
                } else {
                    break;
                }
            }

            // 写法二
//            for (int j = i; j > 0 && array[j].compareTo(array[j - 1]) < 0; j--) {
//                swap(array, j, j - 1);
//            }

            // 写法三
            // 寻找array[i] 合适的插入位置
            Comparable e = array[i];
            int j = i;
            for (; j > 0 && array[j - 1].compareTo(e) > 0; j--) {
                array[j] = array[j - 1];
            }
            array[j] = e;

        }
    }

    /**
     * 调换顺序
     *
     * @param array 进行操作的数组
     * @param i     数组指定位置
     * @param j     数组指定位置
     */
    private static void swap(Object[] array, int i, int j) {
        Object temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }

```

- 写法一

简单直观，寻找array[i]这个元素的合适插入位置，内层循环中，当前位置元素和前一个位置进行比较，如果`array[j]< array[j-1]` 进行两个位置数据交换。调用`swap()`方法.同时调用swap方法时候，是两个元素调换位置，三个变量之间的交换。

- 写法二

在前面提到了，需要进行交换的前提是，`array[j]< array[j-1]` 后一个元素比前一个小。这个从小到大排列违背。我们可以试想可以将这样一个判断条件放入`for`循环当中比较，就有了第二种写法，但是仍然需要进行两个元素调换位置，三个变量之间交换。

- 写法三

可以不用变量，在循环当中将当前元素位置`array[i]`保存起来，然后将`array[j] 和 array[j-1]`交换位置，赋值。


### 运行结果

三种写法，第一种简单直观。下面就来看一看排序的测试用例。

```java
    public static void main(String[] args) {
        Integer[] array = SortTestHelper.generateRandomArray(50000, 0, 100000);
        SortTestHelper.testSort(InsertionSort.class, array);
    }
```

在这里我调用封装的`SortTestHelper`类中的`generateRandomArray`方法生成一个指定长度`n`区间在`[rangeL,rangR]`(0,10000)内的数组，并且调用方法`testSort()`方法对其进行排序，打印其排序50000个元素所花费时间。这里所耗费时间跟测试电脑性能有关。

```java
InsertionSort : 5823 ms

Process finished with exit code 0
```



### 综合测试

上一个例子说到了`选择排序`[算法篇——选择排序SelectionSort](https://ccoder.cc/2017/12/20/selection-sort/)，现在我就来测试一下同一个随机数组使用两个排序算法所耗费时间。

在这里就用到了上面我封装的`SortTestHelper方法`,代码如下

```java
package cc.ccoder.insertion_sort;

import java.lang.reflect.Method;

/**
 * @author : ChenCong
 * @date : Created in 17:22 2017/12/27
 */
public class SortTestHelper {

    private SortTestHelper() {
    }

    /**
     * 生成有n个元素的数组，每个元素的范围是[rangL,rangR]区间
     *
     * @param n      数组大小长度
     * @param rangeL 数组左区间
     * @param rangeR 数组右区间
     * @return 生成的数组
     */
    public static Integer[] generateRandomArray(int n, int rangeL, int rangeR) {
        // 判断数组区间是否合法
        assert rangeL <= rangeR;
        assert n >= 0;
        Integer[] array = new Integer[n];
        for (int i = 0; i < n; i++) {
            //随机产生一个[rangL,rangR]之间的元素，填入数组当中
            array[i] = (int) (Math.random() * ((rangeR - rangeL) + 1)) + rangeL;
        }
        return array;
    }

    /**
     * 打印输出指定数组
     *
     * @param array 将要进行打印的数组
     */
    public static void printArray(Object[] array) {
        for (Object o : array) {
            System.out.print(o + " ");
        }
        System.out.println();
    }

    /**
     * 判断指定数组是否有序 array[i-1] < array[i]
     *
     * @param array 将要进行判断的数组
     * @return 返回是否有序
     */
    public static boolean isSorted(Comparable[] array) {
        for (int i = 0; i < array.length; i++) {
            if (array[i].compareTo(array[i + 1]) > 0) {
                return false;
            }
        }
        return true;
    }

    /**
     * 测试clazz所对应的排序算法排序数组array所得到正确性和算法运行时间
     *
     * @param clazz 将要使用的排序算法
     * @param array 将要进行排序的数组
     */
    public static void testSort(Class<?> clazz, Comparable[] array) {
        try {
            Method sortMethod = clazz.getMethod("sort", Comparable[].class);
            // 参数只有一个，就是将要排序的数组
            Object[] params = new Object[]{array};
            // 开始计数
            Long beginTime = System.currentTimeMillis();
            //调用排序算法
            sortMethod.invoke(null, params);
            //结束计数
            Long endTime = System.currentTimeMillis();
            //判断是否正确排序
            assert isSorted(array);
            //共用时
            System.out.println(clazz.getSimpleName() + " : " + (endTime - beginTime) + " ms");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

- 随机生成指定长度、范围数组`Integer[] generateRandomArray(int n, int rangeL, int rangeR)`

- 判断数组是否有序 `static boolean isSorted(Comparable[] array)`我这里使用从小打到顺序

- 反射调用排序算法，判断其排序是否成功，计算排序时间 `void testSort(Class<?> clazz, Comparable[] array)`

- 打印数组 `void printArray(Object[] array)` 针对少量数据可以调用，数据量上去了不要将数组打印出来。

好了，现在已经封装好了这个类，下面我们就来写一个`Main`来跑一下这两个排序算法。

```java
/**
 * @author : ChenCong
 * @date : Created in 17:10 2017/12/28
 */
public class Main {
    private final static int N = 50000;
    private final static int RANGE_L = 0;
    private final static int RANGE_R = 100000;

    public static void main(String[] args) {
        /**
         * 测试选择排序和插入排序对同一个数组的排序性能比较
         */

        Integer[] arraySelection = SortTestHelper.generateRandomArray(N, RANGE_L, RANGE_R);
        // 对上面的数组进行拷贝一份，切记不能够直接复制，避免两个排序算法对数组进行干扰
        Integer[] arrayInsertion = Arrays.copyOf(arraySelection, arraySelection.length);

        // 开始测试两个排序算法
        SortTestHelper.testSort(InsertionSort.class, arrayInsertion);
        SortTestHelper.testSort(SelectionSort.class, arraySelection);

    }
}
```

跑起来，看一下结果,时间长短和测试机器性能有很大关系。

```java
InsertionSort : 6166 ms
SelectionSort : 4094 ms

Process finished with exit code 0
```

## 后记

### 时间复杂度和空间复杂度

- 插入排序的时间复杂度就是判断比较`array[j] 和 array[j-1]`的次数，那么比较的次数肯定跟原数据的初始顺序有关了，当待排数组有序时候，上述`方法三`中`for (; j > 0 && array[j - 1].compareTo(e) > 0; j--)` 这个for循环的条件就不成立了，不用进入内层循环。此时时间复杂度为`O(N)`。同理，当待排数组为逆序时候，比较次数最大，对于下标为`i`的元素就需要进行`i-1`次比较了，总的次数就为`1+2+3+···+N-1` 此时的时间复杂度为`O(N²)`

- 在上方法当中引入了临时变量，因此空间复杂度为`O(1)`

学习了一下插入排序，根据`插入排序`的这种思想，其实还有`冒泡排序`、`希尔排序`，它们的一些优化、时间复杂度又是怎样的呢？
有时间，再整理整理吧....

### 传送门

- 本次`插入排序`源码部分在github上，可以在下面`联系`当中找到github地址，里面的`algorithm_study`为所有`算法篇源码`

- 直接传送门[`InsertionSort`](https://github.com/chencong-plan/algorithm_study/tree/master/InsertionSort) 

## 联系

[聪聪](https://ccoder.cc/)的独立博客 ，一个喜欢技术，喜欢钻研的95后。如果你看到这篇文章，千里之外，我在等你联系。

- [Blog@ccoder's blog](https://ccoder.cc/)
- [CSDN@ccoder](http://blog.csdn.net/chencong3139)
- [Github@ccoder](https://github.com/chencong-plan)
- [Email@ccoder](mailto:admin@ccoder.top) *or* [Gmail@ccoder](mailto:chencong3139@gmail.com)
