---
layout: post
keywords: blog
description: blog
title: "剑指offer - two day"
tags: [剑指offer]
date: 2017-01-13
categories: 算法
cover: 'https://binarycaptain.github.io/assets/img/suanfa.jpg'
tags: 算法
subtitle: '算法'

---


## 二维数组中的查找

**题目描述**

在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

**解题思路**

从右上角开始查找。因为矩阵中的一个数，它左边的数都比它来的小，下边的数都比它来的大。因此，从右上角开始查找，就可以根据 target 和当前元素的大小关系来改变行和列的下标，从而缩小查找区间。

```java
public boolean find(int target,int array[][]){
    if (array == null && array.length == 0 && array[0].length == 0) return false;
    int m = array.length,n = array[0].length;
    int row = 0,col = n - 1;
    while (row < m && col >= 0 ){
        if (target == array[row][col]) return true;
        else if (target < array[row][col]) row++ ;
        else col--;
    }
    return false;
}
```
