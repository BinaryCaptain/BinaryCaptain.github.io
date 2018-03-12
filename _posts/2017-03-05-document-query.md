---
layout: post
keywords: document
description: document
title: " 详解document.getElementById和document.querySelector的区别"
tags: [document]
date: 2017-03-05
categories: document
cover: 'https://binarycaptain.github.io/assets/img/document.png'
tags: document
subtitle: '关于选择器'

---

## W3C 标准

querySelectorAll 属于 W3C 中的 Selectors API 规范 [1]。而 getElementsBy 系列则属于 W3C 的 DOM 规范 [2]。

在传统的 JavaScript 开发中，查找 DOM 往往是开发人员遇到的第一个头疼的问题，原生的 JavaScript 所提供的 DOM 选择方法并不多，仅仅局限于通过 tag, name, id 等方式来查找，这显然是远远不够的，如果想要进行更为精确的选择不得不使用看起来非常繁琐的正则表达式，或者使用某个库。事实上，现在所有的浏览器厂商都提供了 querySelector 和 querySelectorAll 这两个方法的支持，甚至就连微软也派出了 IE 8 作为支持这一特性的代表，querySelector 和 querySelectorAll 作为查找 DOM 的又一途径，极大地方便了开发者，使用它们，你可以像使用 CSS 选择器一样快速地查找到你需要的节点。

## querySelector

querySelector 和 querySelectorAll 的使用非常的简单，就像标题说到的一样，它和 CSS 的写法完全一样，对于前端开发人员来说，这是难度几乎为零的一次学习。假如我们有一个 id 为 test 的 DIV，为了获取到这个元素，你也许会像下面这样：

>document.getElementById("test");

现在我们来试试使用新方法来获取这个 DIV：

>document.querySelector("#test");
document.querySelectorAll("#test")[0];

获取文档中 class=”example” 的第一个 <p> 元素:

>document.querySelector("p.example");

获取文档中有 “target” 属性的第一个 <a> 元素：

>document.querySelector("a[target]");

假定你选择了两个选择器: <h2> 和 <h3> 元素。以下代码将为文档的第一个 <h2> 元素添加背景颜色：

><h2>A h2 element</h2>
<h3>A h3 element</h3>

document.querySelector("h2, h3").style.backgroundColor = "red";//返回h2或者h3的首个元素

使用这两个方法无法查找带伪类状态的元素，比如querySelector(':hover')不会得到预期结果。

## querySelectorAll

该方法返回所有满足条件的元素，结果是个nodeList集合。查找规则与前面所述一样。

>elements = document.querySelectorAll('div.foo');//返回所有带foo类样式的div

但需要注意的是返回的nodeList集合中的元素是非实时（no-live）的，也就是我们经常所说的静态的，想要区别什么是实时非实时的返回结果，请看下例：

```html
<div id="container">
    <div></div>
    <div></div>
</div>
//首先选取页面中id为container的元素
container=document.getElementById('#container');
console.log(container.childNodes.length)//结果为2
//然后通过代码为其添加一个子元素
container.appendChild(document.createElement('div'));
//这个元素不但添加到页面了，这里的变量container也自动更新了
console.log(container.childNodes.length)//结果为3
```
通过上面的例子就很好地理解了什么是会实时更新的元素。document.getElementById返回的便是实时结果，上面对其添加一个子元素后，再次获取所有子元素个数，已经由原来的2个更新为3个(这里不考虑有些浏览器比如Chrome会把空白也解析为一个子节点)。

感觉区别不大是吧，但如果是稍微复杂点的情况，原始的方法将变得非常麻烦，这时候 querySelector 和 querySelectorAll 的优势就发挥出来了。比如接下来这个例子，我们将在 document 中选取 class 为 test 的 div 的子元素 p 的第一个子元素，当然这很拗口，但是用本文的新方法来选择这个元素，比用言语来描述它还要简单。

>document.querySelector("div.test>p:first-child");
document.querySelectorAll("div.test>p:first-child")[0];

我们使用 querySelectorAll 给所有 class 为 emphasis 的元素加粗显示。

```javascript
var emphasisText = document.querySelectorAll(".emphasis");
for( var i = 0 , j = emphasisText.length ; i < j ; i++ ){
    emphasisText[i].style.fontWeight = "bold";
}
```
这是原生方法，比起jquery速度快。


## document.getElementById和document.querySelector的区别

大部分人都知道，querySelectorAll 返回的是一个 Static Node List，而 getElementsBy 系列的返回的是一个 Live Node List。看看下面这个经典的例子

```javascipt
// Demo 1
var ul = document.querySelectorAll('ul')[0],
    lis = ul.querySelectorAll("li");
for(var i = 0; i < lis.length ; i++){
    ul.appendChild(document.createElement("li"));
}

// Demo 2
var ul = document.getElementsByTagName('ul')[0], 
    lis = ul.getElementsByTagName("li"); 
for(var i = 0; i < lis.length ; i++){
    ul.appendChild(document.createElement("li")); 
}
```
因为 Demo 2 中的 lis 是一个动态的 Node List， 每一次调用 lis 都会重新对文档进行查询，导致无限循环的问题。而 Demo 1 中的 lis 是一个静态的 Node List，是一个 li 集合的快照，对文档的任何操作都不会对其产生影响。但为什么要这样设计呢？其实，在 W3C 规范中对 querySelectorAll 方法有明确规定

>The NodeList object returned by the querySelectorAll() method must be static ([DOM], section 8).

那什么是 NodeList 呢？W3C 中是这样说明的 [7]：

>The NodeList interface provides the abstraction of an ordered collection of nodes, without defining or constraining how this collection is implemented. NodeList objects in the DOM are live

所以，NodeList 本质上是一个动态的 Node 集合，只是规范中对 querySelectorAll 有明确要求，规定其必须返回一个静态的 NodeList 对象。我们再看看在 Chrome 上面是个什么样的情况：

>document.querySelectorAll('a').toString();    // return "[object NodeList]"
document.getElementsByTagName('a').toString();    // return "[object HTMLCollection]"

这里又多了一个 HTMLCollection 对象出来，那 HTMLCollection 又是什么？HTMLCollection 在 W3C 的定义如下:

>An HTMLCollection is a list of nodes. An individual node may be accessed by either ordinal index or the node’s name or id attributes.Note: Collections in the HTML DOM are assumed to be live meaning that they are automatically updated when the underlying document is changed.

实际上，HTMLCollection 和 NodeList 十分相似，都是一个动态的元素集合，每次访问都需要重新对文档进行查询。两者的本质上差别在于，HTMLCollection 是属于 Document Object Model HTML 规范，而 NodeList 属于 Document Object Model Core 规范。这样说有点难理解，看看下面的例子会比较好理解

>var ul = document.getElementsByTagName('ul')[0],
    lis1 = ul.childNodes,
    lis2 = ul.children;
console.log(lis1.toString(), lis1.length);    // "[object NodeList]" 11
console.log(lis2.toString(), lis2.length);    // "[object HTMLCollection]" 4

NodeList 对象会包含文档中的所有节点，如 Element、Text 和 Comment 等。HTMLCollection 对象只会包含文档中的 Element 节点。另外，HTMLCollection 对象比 NodeList 对象 多提供了一个 namedItem 方法。所以在现代浏览器中，querySelectorAll 的返回值是一个静态的 NodeList 对象，而 getElementsBy 系列的返回值实际上是一个 HTMLCollection 对象(动态的nodelist) 。

