---
layout: post
keywords: js
description: js中的选择排序
title: "js中的选择排序"
tags: [js]
date: 2018-08-13
categories: js
cover: 'https://binarycaptain.github.io/assets/img/js.jpg'
tags: js
subtitle: 'js选择排序'
---

## 选择排序

找到数组最小的元素，将它和数组红第一个元素交换位置，接下来，在剩下的元素中找到最小的元素，将它与数组的第二个元素交换位置，往复如此，直到将整个数组排序。基本点就是不断地选择剩余元素之中的最小者。

```javascript

function swap(array,p1,p2) {
	var temp = array[p1];
	array[p1] = array[p2];
	array[p2] = temp;
}

function selectionSort(array) {
	var len = array.length,
		min;

	for(var i=0;i < len;i++) {
		min = i;
		for(j = i+1;j < len;j++){
			if(arr[j] < arr[min]){
				min = j;
			}
		}
		if (i != min){
			swap(array,i,min);
		}
	}
	return array;
}

```








