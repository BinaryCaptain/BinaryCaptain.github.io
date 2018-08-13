---
layout: post
keywords: js
description: js中的 || 和 && 
title: "js中的 || 和 &&"
tags: [js]
date: 2018-08-13
categories: js
cover: 'https://binarycaptain.github.io/assets/img/js.jpg'
tags: js
subtitle: 'js || &&'
---

## javascript “||”、“&&”的灵活运用

你是否看到过这样的代码：a=a||""; 可能javascript初学者会对此感到茫然。今天就跟大家分享一下我的一些心得。 其实：


```javascript

a=a||"defaultValue";


```
与：

```javascript

if(!a){
    a="defaultValue";
}

```

和：

```javascript

if(a==null||a==""||a==undefined){
    a="defaultValue";
}

```

Tips：js中的奇葩假值

1. false
2. null
3. undefined
4. 0
5. '' (空字符串)
6. NaN

是等价的！ 为了弄清这个问题，首先我们必须了解一个问题：javascript中数据类型在转换为bool类型时发生了什么。

在javascript中，数据类型可以分为“真值”和“假值”。顾名思义，真值转换为bool时值为true；假值转换为bool时值为false。下表罗列了一些常见的数据类型转换为bool时的值：


数据类型  转换为bool后的值

null  FALSE

undefined FALSE

Object  TRUE

function  TRUE

0 FALSE

1 TRUE

0、1之外的数字  TRUE

字符串 TRUE

""(空字符串)  FALSE

在if表达式中，javascript首先将条件表达式转换为bool类型，表达式为真值则执行if中的逻辑，否则跳过。

于是有了：

```javascript

if(!a){
    a="defaultValue";
}

```

下面我们再来看“&&”、“||”两个表达式。

由于javascript是弱类型语言，所以在javascript中这两个表达式可能跟其他语言（比如java）中不太一样。

在javascript中

>“&&”运算符运算法则如下：

如果&&左侧表达式的值为真值，则返回右侧表达式的值；否则返回左侧表达式的值。

>“||”运算符的运算法则如下：

如果||左侧表达式的值为真值，则返回左侧表达式的值；否则返回右侧表达式的值。

这2个表达式经常用来做一些简化流程，非常实用！














