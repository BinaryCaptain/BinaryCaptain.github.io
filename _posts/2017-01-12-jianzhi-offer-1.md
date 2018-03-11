---
layout: post
keywords: 算法
description: 算法
title: "剑指offer - one day"
tags: [剑指offer]
date: 2017-01-12
categories: 算法
cover: 'https://binarycaptain.github.io/assets/img/suanfa.jpg'
tags: 算法
subtitle: '算法'

---


## 数组中重复的数字

**题目描述**

在一个长度为 n 的数组里的所有数字都在 0 到 n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。例如，如果输入长度为 7 的数组 {2, 3, 1, 0, 2, 5, 3}，那么对应的输出是第一个重复的数字 2。

**解题思路**

尝试把每个数组元素t放到数组第t个位置上去（简称fix a[t]），若该位置数字已经为t，则出现重复，即不能fix，直接返回t。借助归纳法的思想：
- 当遍历进行到j，j之前的所有位置皆已被fix；
- 处于第j个位置时，只要a[j]不为j，就尝试把a[j]换到a[a[j]]去（即fix a[a[j]]），并循环该过程直到发现重复元素或者正确地fix a[j]；
- 继续遍历直到越界。

```java
public boolean duplicate(int[] numbers,int length,int[] duplication){
    for (int i = 0;i < length; i++) {
        if (numbers == null && length <= 0) return false;
        while (numbers[i] != i && numbers[i] != numbers[numbers[i]]){
            swap(numbers,i,numbers[i])
        }
        if (numbers[i] != i && numbers[i] == numbers[numbers[i]]){
            duplication[0] = numbers[i];
            return true;
        }
}


}
public void swap(int[] numbers,int i,int j){
    int t = numbers[i];
    numbers[i] = numbers[j];
    numbers[j] = t;
}
```
