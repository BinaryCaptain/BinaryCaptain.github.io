---
layout: post
keywords: js数组方法整理
description: 数组方法是否会影响数组本身
title: "数组方法"
tags: [js数组]
date: 2018-07-25
categories: js
tags: js
subtitle: 'js'
cover: 'https://binarycaptain.github.io/assets/img/js.jpg'
---

## 改变原数组

pop()---删除数组的最后一个元素并返回删除的元素。

push()---向数组的末尾添加一个或更多元素，并返回新的长度。

shift()---删除并返回数组的第一个元素。

unshift()---向数组的开头添加一个或更多元素，并返回新的长度。

reverse()---反转数组的元素顺序。

sort()---对数组的元素进行排序。

splice()---用于插入、删除或替换数组的元素。

## 不改变原数组

concat()---连接两个或更多的数组，并返回结果。

every()---检测数组元素的每个元素是否都符合条件。

some()---检测数组元素中是否有元素符合指定条件。

filter()---检测数组元素，并返回符合条件所有元素的数组。

indexOf()---搜索数组中的元素，并返回它所在的位置。

join()---把数组的所有元素放入一个字符串。

toString()---把数组转换为字符串，并返回结果。

lastIndexOf()---返回一个指定的字符串值最后出现的位置，在一个字符串中的指定位置从后向前搜索。

map()---通过指定函数处理数组的每个元素，并返回处理后的数组。

slice()---选取数组的的一部分，并返回一个新数组。

valueOf()---返回数组对象的原始值。

## 方法本质
不改变数组的一些方法如：slice  concat  

都是浅拷贝，浅拷贝的特点是对于引用类型只复制其地址，而不复制真实的值，因此如果操作当中改变引用类型的值，那么真实的值也会发生改变

在使用数组方法时，尽量不要使用改变原数组的一些方法。下面是浅拷贝和深拷贝的简单实现

浅拷贝实现：
```javascript
function shallowCopy(target,source){
	if(!target || typeof target !== 'object'){
		return;
	}

	if(!target || typeof target !== 'object'){
		return;
	}

	for(let key in source){
		if(source.hasOwnProperty(key)){
			target[key] = source[key]
		}
	}
}
```
深拷贝实现：
```javascript
function deepCopy(target,source){
	if(!target || typeof target !=='object'){
		return;
	}

	if(!target || typeof target!== 'object'){
		return;
	}

	for(let key in source){
		if(source.hasOwnProperty(key)) {
			if(source[key] && typeof source[key] == 'object') {
				target[key] = Array.isArray(source[key]) ? []:{};
				deepCopy(target[key],source[key]);
			}else{
				target[key] = source[key]
			}
		}
	}
}
```

