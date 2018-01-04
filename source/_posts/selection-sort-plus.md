---
title: 算法篇——选择排序SelectionSort(泛型比较)
date: 2017-12-20 15:35:01
tags: 
  - SelectionSort
  - 线性结构
  - 选择排序
  - 泛型
categories:
  - 算法学习
---

## 前言

> 上一篇文章[算法篇——选择排序SelectionSort](https://ccoder.cc/2017/12/20/selection-sort/)已经简单的介绍了选择排序，但是存在一个缺陷——在方法`sort`当中参数`array`给的是一个整型，但是如果我想要比较`浮点型`,`字符串`这些对象呢，在实际开发过程当中肯定会根据自定义的规则进行排序，而不是简单的数字大小排序。因此就需要用到接口`Comparable<T>`并且实现其方法`compareTo(Object o)`来自定义规则。

## 场景

### 包装类型及String比较

上面提到了如果有如果存在字符串或者浮点类型该如何比较呢，能否写一个比较通用的方法来解决。这是可以的。
先上`选择排序`的算法吧！
```java
 /**
     * 使用选择排序来进行自定义规则排序
     *
     * @param arr 将要排序的数组
     */
    public static void sort(Comparable[] arr) {
        int n = arr.length;
        for (int i = 0; i < n; i++) {
            int minIndex = i;
            for (int j = i; j < n; j++) {
                //是用CompareTo方法比较两个Comparable对象的大小
                if (arr[j].compareTo(arr[minIndex]) < 0) {
                    minIndex = j;
                }
            }
            swap(arr, i, minIndex);
        }

    }

    /**
     * 对换数组指定索引位置的值
     *
     * @param arr      目标操作数组
     * @param i        源索引
     * @param minIndex 替换索引
     */
    private static void swap(Object[] arr, int i, int minIndex) {
        Object temp = arr[i];
        arr[i] = arr[minIndex];
        arr[minIndex] = temp;
    }
```
在上面`sort()`当中提到了`CompareTo`，在次方法当中的源码中可以看到大小写比较的解释。

```java
    /* @param   o the object to be compared.
     * @return  a negative integer, zero, or a positive integer as this object is less than, equal to, or greater than the specified object.
     *
     * @throws NullPointerException if the specified object is null
     * @throws ClassCastException if the specified object's type prevents it
     *         from being compared to this object.
     */
    public int compareTo(T o);
```

`negative`负数返回小于，`zero`返回`equal`相等，然后`positive`整数放回大于。同时基本数据类型的包装类型都有，还有字符串也实现了这个接口，因此我们对字符串或者浮点型操作时候直接可以使用。操作如下：`main方法当中测试的代码块`

```java
    //测试Integer
    Integer[] a = {10, 9, 8, 7, 6, 5, 4, 3, 2, 1};
    sort(a);
    for (Integer item : a) {
        System.out.print(item + " ");
    }

    System.out.println();  //换行

    // 测试Double
    Double[] d = {6.6,4.5,4.6,7.8,5.8,8.6};
    sort(d);
    for (Double item : d) {
        System.out.print(item+" ");
    }

    System.out.println();

    // 测试String
    String[] s = {"F","B","A","E","D","C"};
    sort(s);
    for (String item : s) {
        System.out.print(item+" ");
    }
```

### 自定义对象比较

#### 需求分析

如果我们对学生`Student`进行排序比较，定义规则`分数socre由小到大，分数相同名字name字典序排列`。正对这样的规则，那么就办法直接使用CompareTo方法了，需要在我们自定义类当中进行实现接口。

- 实现接口 `Comparable<Student>`
- 重写方法 `compareTo(Object o)`
- 定义比较`大小`规则

针对上面的步骤，首先给出`Student`类当中的代码：

```java
package algorithm;

/**
 * @author : ChenCong
 * @date : Created in 15:00 2017/12/20
 */
public class Student implements Comparable<Student> {

    private String name;
    private Integer score;

    public Student(String name, Integer score) {
        this.name = name;
        this.score = score;
    }

    /**
     * 重写Comparable接口的CompareTo方法，实现学生对象的排序
     * 如果分数相等，按照名字字典序排序
     * 如果分数不等，分数低的靠前进行排序
     *
     * @param student 比较student
     * @return  返回结果
     */
    @Override
    public int compareTo(Student student) {
        if (this.score.equals(student.score)) {
            return this.name.compareTo(student.name);
        }
        if (this.score < student.score) {
            return -1;
        } else if (this.score > student.score) {
            return 1;
        } else {
            // this.score == student.score
            return 0;
        }
    }

    /**
     * 重新定义Student实例的打印输出方式
     *
     * @return 放回输出方式
     */
    @Override
    public String toString() {
        return "Student:" + this.name + " " + this.score.toString();
    }
}

```

上面重写`toString()`只是方便打印。

```java 
    if (this.score.equals(student.score)) {
        return this.name.compareTo(student.name);
    }
```

上面`if`语句就是之前提到的规则，如果分数`socre`相等，那么按照名字`name`的字典序进行排序。结果在下面展示。

#### 实现及结果

上面定义`Student`类，下面给出在`main`方法当中使用的结果，及其结果：

```java
        // 测试自定义类型Student
        Student[] students = new Student[5];
        students[0] = new Student("A",88);
        students[1] = new Student("C",70);
        students[2] = new Student("D",95);
        students[3] = new Student("B",90);
        students[4] = new Student("E",88);
        sort(students);
        for (Student item : students) {
            System.out.println(item);
        }

```

在上面的代码当中我们定义了5个`Student`类的对象，同时`A,E`两个分数相同，按照字典序进行排列，应该`A` 在 `E` 的前面，同时两者相挨着。然后分数`socre`是按照从小到大的顺序排列的。

运行上面代码块，显示的结果如下。

```java
整型排序如下：
1 2 3 4 5 6 7 8 9 10 
浮点型排序如下：
4.5 4.6 5.8 6.6 7.8 8.6 
String字符串排序如下：
A B C D E F 
Student对象排序如下:
Student:C 70
Student:A 88
Student:E 88
Student:B 90
Student:D 95
```

## 联系

[聪聪](https://ccoder.cc/)的独立博客 ，一个喜欢技术，喜欢钻研的95后。如果你看到这篇文章，千里之外，我在等你联系。

- [Blog@ccoder's blog](https://ccoder.cc/)
- [CSDN@ccoder](http://blog.csdn.net/chencong3139)
- [Github@ccoder](https://github.com/chencong-plan)
- [Email@ccoder](mailto:admin@ccoder.top) *or* [Gmail@ccoder](mailto:chencong3139@gmail.com)
