---
layout: post
keywords: blog
description: blog
title: "剩余参数和扩展运算符"
tags: [ES6]
date: 2017-02-05
categories: ES6
cover: 'https://binarycaptain.github.io/assets/img/ES6.png'
tags: ES6
subtitle: '不容忽视的...'

---


## 剩余参数
说剩余参数之前，我们首先说一下arguments，arguments对象是所有（非箭头）函数中都可用的局部变量。你可以使用arguments对象在函数中引用函数的参数。此对象包含传递给函数的每个参数的条目，第一个条目的索引从0开始。例如，如果一个函数传递了三个参数，你可以以如下方式引用他们：

>1. arguments[0]
2. arguments[1]
3. arguments[2]

arguments对象不是一个Array它类似于Array，但除了length属性和索引元素之外没有任何Array属性。例如，它没有 pop 方法。

通常我们遍历参数求和的时候，做法如下：
```javascript
    function add(x,y,z){
        return x+y+z;
    }
```
但是当我们的参数变成4个的时候，就无法使用这个函数了，传统的解决方法是利用arguments这个关键字来解决，代码如下：
```javascript
    function add(){
        var sum = 0,
            len = arguments.length;
            for(var i = 0; i<len; i++){
                sum += arguments[i];
            }
            return sum;
    }
add();
add(1,2,3);
add(1,2,3,4);
```
这样我们不管几个参数都可以得到正确的结果。当然ES6为我们提供了更方便的解决方案，那就是剩余参数。那么剩余参数的原型是数组(Array)而不是和arguments一样的类数组对象。有了剩余参数我们可以这样。代码如下：
```javascript
    function add(...numbers){
        return numbers.reduce((prev,curr) => prev+curr),0)
    }
```
### reduce方法介绍
>数组方法 reduce 用来迭代一个数组，并且把它累积到一个值中。
使用 reduce 方法时，你要传入一个回调函数，这个回调函数的参数是一个 累加器 （比如例子中的 prev) 和当前值 (curr）。reduce 方法有一个可选的第二参数，它可以被用来设置累加器的初始值。如果没有在这定义初始值，那么初始值将变成数组中的第一项，而 currentVal 将从数组的第二项开始。
通常我们使用 reduce 方法来让 array 中的所有值相加

这里我们就看到了剩余参数的强大功能。再举一个例子，代码如下：
```javascript
    function converCurrency(rate,...amounts){
        return amounts.map(amounts => amounts * rate);
    }

    const amounts = converCurrency(0.89,12,13,44,32,55);
    console.log(amounts);
```

## 扩展运算符
扩展运算符（spread）是三个点（...）。它好比rest参数的逆运算，将一个数组转为用逗号分隔的参数序列。那么它具体的应用场景有哪些呢，之前我们想合并2个数组，只能这样，代码如下：
```javascript
    const youngers = ["xiaoming","xiaohong","box"];
    const olders = ["ajd","fdjksaf","fdkajf"];
    const members = youngers.concat(olders);
    console.log(members);
```

```javascript
    const youngers = ["xiaoming","xiaohong","box"];
    const olders = ["ajd","fdjksaf","fdkajf"];
    const members = [];
    members = members.concat(youngers);
    members.push("mary");
    members = members.concat(olders);
    console.log(members);
```
这种方法看起来十分臃肿，我们可以使用扩展运算符来解决这个问题。
```javascript
    const youngers = ["xiaoming","xiaohong","box"];
    const olders = ["ajd","fdjksaf","fdkajf"];
    const members = [...youngers,'mary', ...olders];
    console.log(members);
```
之后我们在对扩展运算符的更多用法进行扩展。







