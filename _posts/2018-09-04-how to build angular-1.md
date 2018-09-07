---
layout: post
keywords: react
description: 如何实现一个angular
title: "angular"
tags: [angular]
date: 2018-09-04
categories: angular
cover: 'https://binarycaptain.github.io/assets/img/angular-4.png'
tags: angular
subtitle: '手写一个angular'
---

## $scope实现

AngularJS的$scopes就是一般的JavaScript对象，在它上面你可以绑定你喜欢的属性和其他对象，然而，它们同时也被添加了一些功能用于观察数据结构上的变化。这些观察的功能都由dirty-checking来实现并且都在一个digest循环中被执行。这就是我们在本章中需要实现的功能。

## Scope Objects

我们通过在一个Scope构造函数上面使用new操作符来创建scopes。返回的结果是一个普通的JavaScript对象。我们现在就对这个基本的行为进行测试。

在test / scope_spec.js中为作用域创建测试文件，并将以下测试用例添加到其中：

```javascript

/* jshint globalstrict: true */ 
/* global Scope: false */

'use strict'
describe("Scope", function() {

it("can be constructed and used as an object", function() { var scope = new Scope();
scope.aProperty = 1;
    expect(scope.aProperty).toBe(1);
  });
});

```

在文件的顶部我们启用了ES5的严格模式，同时让JSHint知道我们可以在这个文件中引用一个叫做Scope的全局对象。

这个测试用来创建一个Scope，并在它上面赋一个任意值，然后检查它是否真正被赋值。

>在这里你可能会注意到我们居然使用Scope作为一个全局函数。这绝对不是一个好的JavaScript编程方式！在本书的后面，一旦我们实现了依赖注入，我们将会改正这个错误。

>如果你已经在一个终端中使用了grunt watch，在你添加完这个测试文件之后你会发现它出现了错误，原因在于我们现在还没有实现Scope。而这正是我们想要的，测试驱动开发的第一个重要步骤就是首先要看到错误。 
在本书中我都会假设测试套件会自动执行，同时在测试应该执行时我并不会明确的指出。

我们可以轻松的让这个测试通过：创建src/scope.js文件然后在其中添加以下内容：

```javascript

src/scope.js  
------
/* jshint globalstrict: true */
'use strict'; function Scope() {
}  


```

在这个测试中，我们将一个属性(aProperty)赋值给了这个scope。这正是Scope上的属性运行的方式。它们就是正常的JavaScript属性，并没有什么特别之处。这里你完全不需要去调用一个特别的setter，也不需要对你赋值的类型进行什么限制。真正的魔法在于两个特别的函数：$watch和$digest。我们现在就来看看这两个函数。

## 监视对象属性：$watch和$digest

$watch和$digest是同一个硬币的两面。它们二者同时形成了$digest循环的核心：对数据的变化做出反应。

你可以使用$watch函数为scope添加一个监视器。当这个scope中有变化发生时，监视器便会提醒你。你可以通过给$watch提供两个函数来创建一个监视器：

>作为一个AngularJS用户，你实际上经常指明一个监视表达式而不是一个监视函数。一个监视表达式是一个字符串，例如”user.firstName”，就像你在一个数据绑定，一个指令属性，或者在一段JavaScript代码中指明的那样。它会在AngularJS内部被解析然后编译成一个监视函数。我们将在本书的第二部分中实现这一点。在那之前我们都将使用底层的方法来直接提供一个监视函数。

硬币的另外一面是$digest函数。它迭代了所有绑定到scope中的监视器，然后进行监视并运行相应的监听函数。

为了实现这一块功能，我们首先来定义一个测试文件并断言你可以使用$watch来注册一个监视器，并且当有人调用了$digest的时候监视器的监听函数会被调用。

为了让事情简单一些，我们将在scope_spec.js文件中添加一个嵌套的describe块。并创建一个beforeEach函数来初始化这个scope，以便我们可以在进行每个测试时重复它：








在上面的这个测试中我们调用了$watch来在这个scope上注册一个监视器。我们现在对于监视函数本身并没有什么兴趣，因此我们随便提供了一个函数来返回一个常数值。作为监听函数，我们提供了一个Jasmine Spy。接着我们调用了$digest并检查这个监听器是否真正被调用。




















