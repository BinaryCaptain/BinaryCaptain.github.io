---
layout: post
keywords: blog
description: blog
title: "剑指offer - four day"
tags: [剑指offer]
date: 2017-01-15
categories: 算法
cover: 'https://binarycaptain.github.io/assets/img/suanfa.jpg'
tags: 算法
subtitle: '算法'

---


## 从尾到头打印链表

**解题思路**

正向遍历然后调用 Collections.reverse()。


```java
public String replaceSpace(StringBuffer str) {
	int n = str.length();
	for (int i = 0; i < n; i++){
		if (str.charAt(i) == " ") str.append("  ");
	}
	
	int idxOfOriginal = n - 1;
	int idxOfNew = str.length();
	while( idxOfOriginal >= 0 && idxOfNew > idxOfOriginal) {
		if (str.charAt(i) == ""){
			str.setCharAt(idxOfNew--,"0");
			str.setCharAt(idxOfNew--,"2");
			str.setCharAt(idxOfNew--,"%");
		} else {
			str.setCharAt(idxOfNew--, str.CharAt(idxOfOriginal));
		}
		ideOfOriginal--;
	}
	return str.toString();
}
```
