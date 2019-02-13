---
layout: post
keywords: engine
description: js模板引擎
title: "js模板引擎"
tags: [engine js]
date: 2018-11-09
categories: 模板引擎
cover: 'https://binarycaptain.github.io/assets/img/String-based-Template.png'
tags: engine
subtitle: 'js的几种模板引擎介绍'
---

## 前言

谈起当前前端最热门的 js 框架，必少不了 Vue、React、Angular，对于大多数人来说，我们更多的是在使用框架，对于框架解决痛点背后使用的基本原理往往关注不多，近期在研读 Vue.js 源码，也在写源码解读的系列文章。和多数源码解读的文章不同的是，我会尝试从一个初级前端的角度入手，由浅入深去讲解源码实现思路和基本的语法知识，通过一些基础事例一步步去实现一些小功能。

本场 Chat 是系列 Chat 的开篇，我会首先讲解一下数据双向绑定的基本原理，介绍对比一下三大框架的不同实现方式，同时会一步步完成一个简单的mvvm示例。读源码不是目的，只是一种学习的方式，目的是在读源码的过程中提升自己，学习基本原理，拓展编码的思维方式。

## 模板引擎实现原理

对于页面渲染，一般分为服务器端渲染和浏览器端渲染。一般来说服务器端吐html页面的方式渲染速度更快、更利于SEO，但是浏览器端渲染更利于提高开发效率和减少维护成本，是一种相关舒服的前后端协作模式，后端提供接口，前端做视图和交互逻辑。前端通过Ajax请求数据然后拼接html字符串或者使用js模板引擎、数据驱动的框架如Vue进行页面渲染。

在ES6和Vue这类框架出现以前，前端绑定数据的方式是动态拼接html字符串和js模板引擎。模板引擎起到数据和视图分离的作用，模板对应视图，关注如何展示数据，在模板外头准备的数据， 关注那些数据可以被展示。模板引擎的工作原理可以简单地分成两个步骤：模板解析 / 编译（Parse / Compile）和数据渲染（Render）两部分组成，当今主流的前端模板有三种方式：

1. String-based templating (基于字符串的parse和compile过程)

2. Dom-based templating (基于Dom的link或compile过程)

3. Living templating (基于字符串的parse 和 基于dom的compile过程)

## String-based templating

基于字符串的模板引擎，本质上依然是字符串拼接的形式，只是一般的库做了封装和优化，提供了更多方便的语法简化了我们的工作。基本原理如下：

![](https://binarycaptain.github.io/assets/img/String-based-Template.png)

典型的库：

1. art-template

2. mustache.js

3. doT

之前的一篇文章中我介绍了js模板引擎的实现思路，感兴趣的朋友可以看看这里：JavaScript进阶学习（一）—— 基于正则表达式的简单js模板引擎实现。这篇文章中我们利用正则表达式实现了一个简单的js模板引擎，利用正则匹配查找出模板中{{}}之间的内容，然后替换为模型中的数据，从而实现视图的渲染。

```javascript

var template = function(tpl, data) {
  var re = /{{(.+?)}}/g,
  cursor = 0,
  reExp = /(^( )?(var|if|for|else|switch|case|break|{|}|;))(.*)?/g,
  code = 'var r=[];\n';
  // 解析html
  function parsehtml(line) {
    // 单双引号转义，换行符替换为空格,去掉前后的空格
    line = line.replace(/('|")/g, '\\$1').replace(/\n/g, ' ').replace(/(^\s+)|(\s+$)/g,"");
    code +='r.push("' + line + '");\n';
  }

  // 解析js代码		
  function parsejs(line) {   
    // 去掉前后的空格
    line = line.replace(/(^\s+)|(\s+$)/g,"");
    code += line.match(reExp)? line + '\n' : 'r.push(' + 'this.'+line + ');\n';
  }	

  while((match = re.exec(tpl))!== null) {
    // 开始标签  {{ 前的内容和结束标签 }} 后的内容
    parsehtml(tpl.slice(cursor, match.index))
    // 开始标签  {{ 和 结束标签 }} 之间的内容
    parsejs(match[1])
    // 每一次匹配完成移动指针
    cursor = match.index + match[0].length;
  }
  // 最后一次匹配完的内容
  parsehtml(tpl.substr(cursor, tpl.length - cursor));
  code += 'return r.join("");';
  **return new Function(code.replace(/[\r\t\n]/g, '')).apply(data);**
}

var tpl = document.getElementById("template").innerHTML.toString();
document.getElementById("content").innerHTML = template(tpl,{
  name: 'zhaomenghuan',
  profile: { 
    age: 24 
  },
  sex: 'man',
  skills: ['html5', 'javascript', 'android']
});
}

```

现在ES6支持了模板字符串，我们可以用比较简单的代码就可以实现类似的功能：

```javascript

const template = data => `
	<p>name: ${data.name}</p>
	<p>age: ${data.profile.age}</p>
	<ul>
		${data.skills.map(skill => `
			<li>${skill}</li>
		`).join('')}
	</ul>`

const data = {
	name:'yanchao',
	profile:{age:23},
	skills:['html5','javascript','android']
}

document.body.innerHTML = template(data)

```

## Dom-based templating

Dom-based templating 则是从DOM的角度去实现数据的渲染，我们通过遍历DOM树，提取属性与DOM内容，然后将数据写入到DOM树中，从而实现页面渲染。一个简单的例子如下：

```javascript

function MVVM(opt) {
  this.dom = document.querySelector(opt.el);
  this.data = opt.data || {};
  this.renderDom(this.dom);
}

MVVM.prototype = {
  init: {
    sTag: '{{',
    eTag: '}}'
  },
  render: function (node) {
    var self = this;
    var sTag = self.init.sTag;
    var eTag = self.init.eTag;

    var matchs = node.textContent.split(sTag);
    if (matchs.length){
      var ret = '';
      for (var i = 0; i < matchs.length; i++) {
        var match = matchs[i].split(eTag);
        if (match.length == 1) {
            ret += matchs[i];
        } else {
            ret = self.data[match[0]];
        }
        node.textContent = ret;
      }
    }
  },
  renderDom: function(dom) {
    var self = this;

    var attrs = dom.attributes;
    var nodes = dom.childNodes;

    Array.prototype.forEach.call(attrs, function(item) {
      self.render(item);
    });

    Array.prototype.forEach.call(nodes, function(item) {
      if (item.nodeType === 1) {
        return self.renderDom(item);
      }
      self.render(item);
    });
  }
}

var app = new MVVM({
  el: '#app',
  data: {
    name: 'zhaomenghuan',
    age: '24',
    color: 'red'
  }
});


```

页面渲染的函数 renderDom 是直接遍历DOM树，而不是遍历html字符串。遍历DOM树节点属性（attributes）和子节点（childNodes），然后调用渲染函数render。当DOM树子节点的类型是元素时，递归调用遍历DOM树的方法。根据DOM树节点类型一直遍历子节点，直到文本节点。

render的函数作用是提取{{}}中的关键词，然后使用数据模型中的数据进行替换。我们通过textContent获取Node节点的nodeValue，然后使用字符串的split方法对nodeValue进行分割，提取{{}}中的关键词然后替换为数据模型中的值。

DOM 的相关基础

注：元素类型对应NodeType

元素类型  NodeType
  元素      1
  属性      2
  文本      3
  注释      8
  文档      9

childNodes 属性返回包含被选节点的子节点的 NodeList。childNodes包含的不仅仅只有html节点，所有属性，文本、注释等节点都包含在childNodes里面。children只返回元素如input, span, script, div等，不会返回TextNode，注释。
















