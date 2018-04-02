---
layout: post
keywords: interview
description: interview
title: "有赞面试题"
tags: [interview]
date: 2018-03-18
categories: interview
cover: 'https://binarycaptain.github.io/assets/img/youzan.jpg'
tags: interview
subtitle: 'interview'

---


## 什么是 Event Emitter

Event emitter 听起来只是触发一个事件，这个事件任何东西都能监听。

想象一下这样的场景，在你的异步代码中，去“呼叫”一些事件的发生，以及让你其他部分都要听到你的“呼叫”并且注册他们的想法。

为了不同的目的，对于 Event Emitter 模式有大量不同的实现，但是基本的想法是为了给一个框架提供事件的管理以及能够去订阅他们。

在这里，我们的目标创建属于我们自己的 Event Emitter 去理解背后的秘密。所以，让我们看一下下面的代码是怎么工作的。

```javascript
let input = document.querySelector('input[type="text"]');
let button = document.querySelector('button');
let h1 = document.querySelector('h1');

button.addEventListener('click', () => {
    emitter.emit('event:name-changed', { name: input.value });
});

let emitter = new EventEmitter();
emitter.subscribe('event:name-changed', data => {
    h1.innerHTML = `Your name is: ${data.name}`;
});
```

让我们开始。

```javascript
class EventEmitter() {
	constructor() {
		this.events = {};
	}
}
```

我们先创建一个 EventEmiiter 类以及初始化 events 空对象属性。这个 events 属性的目的是为了存储我们的事件集合，这个 events 对象使用事件名当做key，用订阅者集合当做value。（可以把每个订阅者看作是一个函数）。

订阅函数

```javascript
subscribe(eventName, fn) {
    if (!this.events[eventName]) {
        this.events[eventName] = [];
    }

    this.events[eventName].push(fn);
}
```

这个订阅函数获取事件名称，在我们之前的例子中，它是 "event:name-changed" 以及传入一个回调，当有人调用 emit（或尖叫）事件的时候调用回调。

在 JavaScript 函数的优点之一是函数是第一对象，所以我们能像之前我们的订阅方法一样，通过函数作为另一个函数的参数。

如果未注册这个事件，我们需要在第一次为它设置一个初始值，事件名称作为 key 以及初始化一个空数组赋值给它，然后我们将函数放入这个数组，以便我们想通过 emit 去调用这个事件。

## 调用函数

```javascript
emit(eventName, data) {
    const event = this.events[eventName];
    if (event) {
        event.forEach(fn => {
            fn.call(null, data);
        });
    }
}
```

这个调用函数接受事件名，这个事件名是我们想“呼叫”的名称，以及我们想传递给这个事件的数据。如果在我们的 events 中存在这个事件，我们将带上数据循环调用所有订阅的方法。

使用上面的代码能做我们所说的全部的事情。但我们仍然有一个问题。当我们不再需要它们的时候，我们需要一种方法来取消注册这些订阅，因为如果你不这样做，将造成内存泄漏。

让我们来解决这个问题，通过在订阅函数中返回一个取消注册的方法。

```javascript
subscribe(eventName, fn) {
    if (!this.events[eventName]) {
        this.events[eventName] = [];
    }

    this.events[eventName].push(fn);

    return () => {
        this.events[eventName] = this.events[eventName].filter(eventFn => fn !== eventFn);
    }
}
```

因为 JavaScript 函数是第一对象，你能在一个函数中返回一个函数。因此现在我们能调用这个取消注册函数，如下：

```javascript
let unsubscribe = emitter.subscribe('event:name-changed', data => console.log(data));

unsubscribe();
```

当我们调用取消注册函数的时候，我们删除的功能依赖于对订阅函数集合的筛选方法（Array filter）。

和内存泄露说再见！

## 实际案列

```html
<!DOCTYPE html>
<html>

<head>
    <script src="script.js"></script>
</head>

<body>
    <input type="text">
    <h1></h1>
    <button>Change name</button>
</body>
</html>
```
```javascript
    class EventEmitter {
        constructor() {
            this.events = {};
        }

    subscribe(eventName,fn) {
            if (!events[eventName]) {
                this.events[eventName] = [];
            }

            this.events[eventName].push(fn);

            return this.events[eventName] = this.events[eventName].filter(eventFn => fn !== eventFn);
        }

    }

    emit(eventName,data) {
        const event = this.events[eventName];
        event.forEach(fn => {
            fn.call(null,data);
        })
    }

    document.addEventListener("DOMContentLoaded",function(event) {

        })


```











