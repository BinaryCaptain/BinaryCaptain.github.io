---
layout: post
keywords: js
description: js中的二分查找
title: "js中的二分查找"
tags: [js]
date: 2018-08-13
categories: js
cover: 'https://binarycaptain.github.io/assets/img/js.jpg'
tags: js
subtitle: 'js二分查找'
---

## 二分查找

二分法查找，也称折半查找，是一种在**有序数组**中查找特定元素的搜索算法。查找过程可以分为以下步骤：
（1）首先，从有序数组的中间的元素开始搜索，如果该元素正好是目标元素（即要查找的元素），则搜索过程结束，否则进行下一步。
（2）如果目标元素大于或者小于中间元素，则在数组大于或小于中间元素的那一半区域查找，然后重复第一步的操作。
（3）如果某一步数组为空，则表示找不到目标元素。

二分的写法有递归和非递归2种写法

#### 非递归写法

```javascript

function binarySearch(arr,target) {
	var low = 0;
		high = arr.length -1;
	while(low <= high) {
		var mid = low + (high-low)/2;
		if(mid == target) {
			return mid;
		}else if(target > arr[mid]){
			low = mid + 1;
		}else if(target < arr[mid]){
			high = mid -1;
		}else{
			return -1;
		}
	}
}

```

#### 递归写法

```javascript

function binarySearch(arr,low,high,target) {
	if(low > high) {
		return -1;
	}
	var mid = low + (high-low)/2;
	if(arr[mid] == target) {
		return mid;
	}else if(target < arr[mid]) {
		high = mid - 1;
		return binarySearch(arr,low,high,target);
	}else if(target > arr[mid]) {
		low = mid + 1;
		return binarySearch(arr,low,high,target);
	}
}

```







