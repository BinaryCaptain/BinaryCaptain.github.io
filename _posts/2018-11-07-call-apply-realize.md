---
layout: post
keywords: call apply
description: 用原生方法实现call和apply 
title: "原生方法实现call和apply"
tags: [call apply]
date: 2018-11-07
categories: call apply
cover: 'https://binarycaptain.github.io/assets/img/js.jpg'
tags: call apply
subtitle: '原生方法实现call和apply(低于ES6)'
---

## call的实现

>call()方法在使用一个指定的this值和若干个指定的参数值得前提下调用某个函数或方法。

举个列子：

```javascirpt

let foo = {
	value: 1
}

function bar() {
	console.log(this.value)
}

bar.call(foo) // 1

```

注意两点：

1. call 改变了 this 的指向，指向到 foo
2. bar 函数执行了

## 模拟实现第一步

那么我们该怎么模拟实现这两个效果呢？

试想当调用 call 的时候，把 foo 对象改造成如下：

```javascript

let foo = {
	value:1,
	bar:function() {
		console.log(this.value)
	}
}

foo.bar(); // 1

```

这个时候 this 就指向了 foo，是不是很简单呢？

但是这样却给 foo 对象本身添加了一个属性，这可不行呐！

不过也不用担心，我们用 delete 再删除它不就好了~

所以我们模拟的步骤可以分为：

将函数设为对象的属性
执行该函数
删除该函数
以上个例子为例，就是：

```javascipt

// 第一步
foo.fn = bar
// 第二步
foo.fn()
// 第三步
delete foo.fn

```

fn 是对象的属性名，反正最后也要删除它，所以起成什么都无所谓。

根据这个思路，我们可以尝试着去写第一版的 call2 函数：

```javascript

Function.prototype.call2 = function(context) {
	//获取调用call的函数，用this可以获取
	context.fn = this;
	context.fn()
	delete context.fn
}

let foo = {
	value: 1
}

function bar() {
	console.log(this.value)
}

bar.call2(foo);

```

## 模拟实现第二步

最一开始也讲了，call 函数还能给定参数执行函数。举个例子：

```javascript

let foo = {
    value: 1
};

function bar(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value);
}

bar.call(foo, 'kevin', 18);
// kevin
// 18
// 1

```

注意：传入的参数并不确定，这可咋办？

不急，我们可以从 Arguments 对象中取值，取出第二个到最后一个参数，然后放到一个数组里。

比如这样：

```javascript

// 以上个例子为例，此时的arguments为：
// arguments = {
//      0: foo,
//      1: 'kevin',
//      2: 18,
//      length: 3
// }
// 因为arguments是类数组对象，所以可以用for循环
var args = [];
for(var i = 1, len = arguments.length; i < len; i++) {
    args.push('arguments[' + i + ']');
}

// 执行后 args为 ["arguments[1]", "arguments[2]", "arguments[3]"]


```

不定长的参数问题解决了，我们接着要把这个参数数组放到要执行的函数的参数里面去。

```javascript

// 将数组里的元素作为多个参数放进函数的形参里
context.fn(args.join(','))
// (O_o)??
// 这个方法肯定是不行的啦！！！

```

也许有人想到用 ES6 的方法，不过 call 是 ES3 的方法，我们为了模拟实现一个 ES3 的方法，要用到ES6的方法，好像……，嗯，也可以啦。但是我们这次用 eval 方法拼成一个函数，类似于这样：

```
eval('context.fn(' + args + ')')

```

这里 args 会自动调用 Array.toString() 这个方法。

所以我们的第二版克服了两个大问题，代码如下：

```javascript

Function.prototype.call2 = function(context) {
	content.fn = this;
	let args = []
	for(let i = 0;i < arguments.length;i++) {
		args.push('arguments[' + i ']');
	}
	eval('context.fn(' + args + ')');
	delete context.fn 
}

// 测试一下
var foo = {
    value: 1
};

function bar(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value);
}

bar.call2(foo, 'kevin', 18); 
// kevin
// 18
// 1

```

## 模拟实现第三步

模拟代码已经完成 80%，还有两个小点要注意：

1. this 参数可以传 null，当为 null 的时候，视为指向 window

举个例子：

```javascript

var value = 1;

function bar() {
    console.log(this.value);
}

bar.call(null); // 1

```

虽然这个例子本身不使用 call，结果依然一样。

2. 函数是可以有返回值的！

举个例子：

```javascipt

var obj = {
    value: 1
}

function bar(name, age) {
    return {
        value: this.value,
        name: name,
        age: age
    }
}

console.log(bar.call(obj, 'kevin', 18));
// Object {
//    value: 1,
//    name: 'kevin',
//    age: 18
// }


```

不过都很好解决，让我们直接看第三版也就是最后一版的代码：


```javascript

// 第三版
Function.prototype.call2 = function (context) {
    var context = context || window;
    context.fn = this;

    var args = [];
    for(var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }

    var result = eval('context.fn(' + args +')');

    delete context.fn
    return result;
}

// 测试一下
var value = 2;

var obj = {
    value: 1
}

function bar(name, age) {
    console.log(this.value);
    return {
        value: this.value,
        name: name,
        age: age
    }
}

bar.call2(null); // 2

console.log(bar.call2(obj, 'kevin', 18));
// 1
// Object {
//    value: 1,
//    name: 'kevin',
//    age: 18
// }

```

## apply的模拟实现

apply 的实现跟 call 类似，在这里直接给代码，代码来自于知乎 @郑航的实现：

```javascript

Function.prototype.apply = function(context,arr) {
	let context = Object(context) || window;
	context.fn = this;

	let result;
	if (!arr) {
		result = context.fn()
	}else{
		let args = []
		for(let i = 0;i < arr.length;i++) {
			args.push('arr[' + i + ']')
		}
		result = eval('context.fn(' + args + ')')
	}
	delete context.fn
	return result
}

```













