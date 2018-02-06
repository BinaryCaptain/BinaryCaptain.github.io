---
layout: post
keywords: blog
description: blog
title: "如何实现一个简单的Virtual DOM算法"
---
{% include codepiano/setup %}

看到一篇关于Virtual DOM的优秀文章，现转载

###1 前言

本文会在教你怎么用 300~400 行代码实现一个基本的 Virtual DOM 算法，并且尝试尽量把 Virtual DOM 的算法思路阐述清楚。希望在阅读本文后，能让你深入理解 Virtual DOM 算法，给你现有前端的编程提供一些新的思考。

###2 对前端应用状态管理的思考

假如现在你需要写一个像下面一样的表格的应用程序，这个表格可以根据不同的字段进行升序或者降序的展示。

![](/image/table.png)

这个应用程序看起来很简单，你可以想出好几种不同的方式来写。最容易想到的可能是，在你的 JavaScript 代码里面存储这样的数据：

``` ruby
var sortKey = "new" // 排序的字段，新增（new）、取消（cancel）、净关注（gain）、累积（cumulate）人数
var sortType = 1 // 升序还是逆序
var data = [{...}, {...}, {..}, ..] // 表格数据
```

其中 *$ACTIVITY$* 代表当前所在的类名，*$CURSOR$* 代表当前鼠标的定位位置，同理 *newInstance* 可以帮你在 Fragment 中快速声明一个新建 Fragment 的方法，它的代码如下：

``` ruby
public static $fragment$ newInstance($args$) {
    $nullChecks$
    Bundle args = new Bundle();
    $addArgs$
    $fragment$ fragment = new $fragment$();
    fragment.setArguments(args);
    return fragment;
}
```
其中 *$$* 代表是一个变量，中间包裹着这个变量的名字，你可以对这个变量声明类型，这个后面再说。

是不是很容易理解呢？如果理解了那么就可以来根据自己的使用习惯来定义自己的 *Live Templates* 了。

比如我们在开发中要经常写单例模式吧？每次都要写这么一大段是不是很烦？那么今天就教大家自定义一个单例模式的模板，以后轻松搞定单例。

到 *设置 -> Editor -> Live Templates* ，点击右上角的 + 号，选择 Template Group ，因为我习惯自定义的单独分组先，这样好管理，比如新建一个分组叫做 *stormzhang* ，然后就会看到有一个 *stormzhang* 的分组显示在了列表里，这时候鼠标选中该分组，然后再点击右上角的 + 号，点击 *Live Template* ，然后如下图填写缩写与描述，紧接着把如下代码拷贝到下面的输入框里（PS：单例模式的写法有很多种，这里就随意以其中一种为例）

``` ruby
private static $CLASS$ instance = null;
 
private $CLASS$(){
}
 
public static $CLASS$ getInstance() {
    synchronized ($CLASS$.class) {
        if (instance == null) {
            instance = new $CLASS$();
        }
    }

    return instance;
}
```
注意这里，如果你这段代码是一些固定的代码，那么至此就结束了，但是这段代码里是动态的，里面有一些变量，因为每个类的类名如果都需要自己手动更改就太麻烦了，所以有个变量 *$CLASS$* ，所以需要点击下面的 *Define* ，先要定义变量所属的语言范围，点开之后可以看到这里支持 HTML、XML、JSON、Java、C++ 等，很明显，我们这里需要支持 Java ，选择选中 Java :

![](/image/live_templates3.png)

紧接着，我们需要给变量 *$CLASS$* 定义类型，这里的 *CLASS* 名字随意取的，为了可读性而已，你高兴可以取名 *abc* ，真正给这个变量定义类型的是点击 *Edit variables* 按钮，来对该变量进行编辑，我们选择 *className()* 选项，可以看到还有其他选项，但是看名字大家大概就猜到什么含义了，这里就不一一解释了。

![](/image/live_templates4.png)

点击 *ok* 保存，至此我们定义的一个单例的 Live Template 就完成了。你可以随意打开一个类文件，然后输入 *singleton* 按 *tab* 或者 *enter* 键就可以看到神奇的一幕出现了，是不是很帅？

看完这篇文章想想自己还有哪些常用到的代码片段，赶紧把它定义成一个 Live Template 吧，你会发现你又可以变懒了！



