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

public ArrayList<Integer> printListFromTainToHead(ListNode listNode) {
	ArrayList<Integer> ret = new ArrayList<>();
	if (listNode != null) {
		ret.add(listNode.val);
		listNode = listNode.next;
	}
	Collections.reverse(ret);
	return ret;
}

使用 Stack
public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
	Stack<Integer> stack = new Stack<>();
	if (listNode != null) {
		stack.add(listNode.val);
		listNode = listNode.next;
	}
	ArrayList<Integer> ret = new ArrayList<>();
	while (!stack.isEmpty()) {
		ret.add(stack.pop());
	}
	return ret;
}

使用递归
public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
	ArrayList<Integer> ret = new ArrayList<>();
	if (listNode != null) {
		ret.addAll(printListFromTailToHead(listNode.next));
		ret.add(listNode.val);
	}
	return ret;
}

不使用库函数，并且不使用递归的迭代实现，利用链表的头插法为逆序的特性。



```
