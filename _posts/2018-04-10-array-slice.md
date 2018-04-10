---
layout: post
keywords: 数组
description: 数组方法
title: "slice"
tags: [数组]
date: 2018-03-18
categories: 数组
cover: 'https://binarycaptain.github.io/assets/img/js.jpg'
tags: 数组
subtitle: '数组'

---


## js中的slice方法

### String.slice(start,end)
slice（）返回一个子片段，对原先的string没有影响,与subString的区别是，还可以用负数当参数，相当于是length+start,length+end.

```javascript
var s = "abcdefg";
s.slice(0,4)    // Returns "abcd"
s.slice(2,4)    // Returns "cd"
s.slice(4)      // Returns "efg"
s.slice(3,-1)   // Returns "def"
s.slice(3,-2)   // Returns "de"
s.slice(-3,-1)  // Should return "ef"; returns "abcdef" in IE 4
```

### Array.slice(start,end)

返回从start开始到end的子数组，如果end这个参数没有被设置，则返回从start开始到最后的数组元素。

```javascript
var a = [1,2,3,4,5];
a.slice(0,3);    // Returns [1,2,3]
a.slice(3);      // Returns [4,5]
a.slice(1,-1);   // Returns [2,3,4]
a.slice(-3,-2);  // Returns [3]; buggy in IE 4: returns [1,2,3]
```

除了正常用法，slice 经常用来将 array-like 对象转换为 true array。在一些框架中会经常有这种用法。

Array.prototype.slice.call(arguments);//将参数转换成真正的数组.

因为arguments不是真正的Array,虽然arguments有length属性，但是没有slice方法，所以呢，Array.prototype.slice（）执行的时候，Array.prototype已经被call改成arguments了，因为满足slice执行的条件(有length属性).

## 涉及到的一道面试题

统计页面上所有的HTML标签，有多少种，每种出现了多少次，按照出现次数进行倒序

```javascript
let nodeList = document.querySelectorAll("*") || [];
nodeList = Array.prototype.slice.call(nodeList);
nodeList = nodeList.map((e) => e.tagName);

let uniqueArr = [];
let frequenceObj = {};
for (let i = 0; i < nodeList.length; i++) {
  if (uniqueArr.indexOf(nodeList[i]) > -1) {
    frequenceObj[nodeList[i]]++;
  } else {
    let frequenceName = nodeList[i];
    uniqueArr.push(frequenceName);
    frequenceObj[frequenceName] = 1;
  }
}
// 根据得到的去重数组排序
let resultArr = [];
for (i in frequenceObj) {
  let obj = {
    name: i,
    frequence: frequenceObj[i]
  }
  resultArr.push(obj)
}
resultArr.sort((x, y) => y.frequence - x.frequence)
console.log(resultArr);

```




