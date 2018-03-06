---
layout: post
keywords: blog
description: blog
title: "剑指offer - three day"
tags: [剑指offer]
date: 2017-01-14
categories: 算法
cover: 'https://binarycaptain.github.io/assets/img/suanfa.jpg'
tags: 算法
subtitle: '算法'

---


## 替换空格

**题目描述**
请实现一个函数，将一个字符串中的空格替换成“%20”。例如，当字符串为 We Are Happy. 则经过替换之后的字符串为 We%20Are%20Happy。

**题目要求**
以 O(1) 的空间复杂度和 O(n) 的空间复杂度来求解。

**解题思路**

从后向前改变字符串。


```java
public String replaceSpace(StringBuffer str) {
    int n = str.length();
    for (int i = 0; i < n; i++) {
       if (str.charAt(i) == ' ') str.append("  ") // 尾部填充两个
    }

    int idxOfOriginal = n - 1;  //长度为n，索引为0 - n-1
    int idxOfNew = str.length() - 1;
    while (idxOfOriginal >= 0 && idxOfNew > idxOfOriginal) {
        if (str.charAt(idxOfOriginal) == ' ') {
            str.setCharAt(idxOfNew--, '0');
            str.setCharAt(idxOfNew--, '2');
            str.setCharAt(idxOfNew--, '%');
        } else {
            str.setCharAt(idxOfNew--, str.charAt(idxOfOriginal));
        }
        idxOfOriginal--;
    }
    return str.toString();
}
```
