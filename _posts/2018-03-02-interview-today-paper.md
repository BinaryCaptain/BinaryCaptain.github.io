---
layout: post
keywords: interview
description: interview
title: " 今日头条面试题"
tags: [interview]
date: 2018-03-18
categories: interview
cover: 'https://binarycaptain.github.io/assets/img/first-paper.jpeg'
tags: interview
subtitle: 'interview'

---


## 关于链表的一道算法题目

给定一个乱序的链表，现在有一个操作：可以把链表任意位置的值移动到链表的最后。求链表排序所需要的最少操作次数。
样例：
假如链表的值为：5 1 2 3 那么最少的操作次数为1，直接把5移到最后即可，输出为1.
再比如 链表值为 5 2 1 3 ，那么应该先把2移到最后，再把3移到最后，再把5移到最后，这样输出的结果为3.

这里只说一下思路：

把链表所有值扔到一个最小堆，count计数为0，遍历链表找到和最小堆堆顶元素相等的节点，最小堆出堆，count计数+1，检测当前堆顶是否和下一个节点相等，相等的话重复上述操作，不相等说明链表中从最小节点开始符合全局有序的序列长度为count，需要移动的最小次数是总数n-count
如5 2 1 3 ，第一次堆顶为1，找到链表中1的位置，下一个堆顶元素为2，但是节点的下一个元素为3，不相等，所以最小移动次数为4-1=3。

果然是宇宙条题目真的难