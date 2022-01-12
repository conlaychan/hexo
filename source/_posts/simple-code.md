---
title: 算法：冒泡排序，二分查找，约瑟夫环
date: 2022-01-10
tags:
  - 算法
categories: java
description: Java 实现：冒泡排序，二分查找，约瑟夫环
---

## 冒泡排序

> 把每个数字与其后面的每个数字依次比较，若后面的比前面的小，就交换位置。

```java
public static void bubbleSort(int[] array) {
    for (int i = 0; i < array.length; i++) {
        for (int j = i + 1; j < array.length; j++) {
            if (array[j] < array[i]) {
                int temp = array[j];
                array[j] = array[i];
                array[i] = temp;
            }
        }
    }
}
```

## 二分查找

> 从一个<font color="red">**有序数组**</font>中找出一个特定值的下标号，用数组中间位置的值与目标值比较大小，相等则视为找到，若目标值小于中间值则再从数组前半段以同样方式查找，若目标值大于中间值则再从数组后半段以同样方式查找。
>
> <font color="red">**注意前提：数组是有序的，我以升序为例**</font>。

```java
public static int binarySearch(int[] array, int target) {
    if (array == null || array.length == 0) {
        throw new IllegalArgumentException("数组中不存在目标数值");
    }
    Arrays.sort(array);

    int fromIndex = 0;
    int toIndex = array.length - 1;
    while (fromIndex <= toIndex) {
        int middleIndex = (fromIndex + toIndex) / 2;
        int middleValue = array[middleIndex];
        if (middleValue == target) {
            return middleIndex;
        } else if (target < middleValue) {
            toIndex = middleIndex - 1;
        } else {
            fromIndex = middleIndex + 1;
        }
    }
    throw new IllegalArgumentException("数组中不存在目标数值");
}
```

## 约瑟夫环

> 队列中有n个人，依次喊号码1-2-3-1-2-3……喊到3的人退出队列，最后一个人喊完后就从第一个人接着喊（即最后一个人喊2时回到第一个人去喊3），直到队列中只剩一人，推算剩下的这一个人是队列中最开始时的第几个人。

```java
public static int leaveOne(int total) {
    if (total < 1) {
        throw new IllegalArgumentException();
    }
    boolean[] out = new boolean[total]; // false：还在队列中，true：已离开队列
    int left = total; // 剩余人数
    int number = 0; // 喊出号码
    
    while (left > 1) {
        for (int i = 0; i < out.length; i++) {
            if (out[i]) {
                continue;
            }
            number++; // 喊号
            if (number == 3) {
                number = 0;
                out[i] = true; // 出局
                left--;
            }
        }
    }
    for (int i = 0; i < out.length; i++) {
        if (!out[i]) {
            return i + 1; // 数组下标号加1
        }
    }
    throw new AssertionError("总人数为0"); // 这一行永远不可能执行到
}
```

------
![福利](/images/骚图/三国杀/双乔2.jpg)
