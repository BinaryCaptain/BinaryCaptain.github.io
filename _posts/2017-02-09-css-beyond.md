---
layout: post
keywords: blog
description: blog
title: "css溢出处理"
tags: [css溢出]
date: 2017-02-09
categories: css
cover: 'https://binarycaptain.github.io/assets/img/beyond-1.png'
tags: css
subtitle: '关于css溢出处理'

---


## css实现单行文本溢出显示 ...

直接上效果：相对于多行文本溢出做处理， 单行要简单多，且更容易理解。

![](https://binarycaptain.github.io/assets/img/beyond-1.png)

实现方法

```css
overflow: hidden;
text-overflow:ellipsis;
white-space: nowrap;
```
当然还需要加宽度width属来兼容部分浏览。

## 实现多行文本溢出显示...

效果：

![](https://binarycaptain.github.io/assets/img/beyond-2.png)

实现方法：

```css
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 3;
overflow: hidden;
```
适用范围：

因使用了WebKit的CSS扩展属性，该方法适用于WebKit浏览器及移动端；

注：

>1、-webkit-line-clamp用来限制在一个块元素显示的文本的行数。 为了实现该效果，它需要组合其他的WebKit属性。常见结合属性：
2、display: -webkit-box; 必须结合的属性 ，将对象作为弹性伸缩盒子模型显示 。
3、-webkit-box-orient 必须结合的属性 ，设置或检索伸缩盒对象的子元素的排列方式 。

如果你觉着这样还不够美观， 那么就接着往下看：

效果：

![](https://binarycaptain.github.io/assets/img/beyond-3.png)

这样 是不是你想要的呢？

实现方法：

```html
    div {
    position: relative;
    line-height: 20px;
    max-height: 40px;
    overflow: hidden;
}

div:after {
    content: "..."; position: absolute; bottom: 0; right: 0; padding-left: 40px;
    background: -webkit-linear-gradient(left, transparent, #fff 55%);
    background: -o-linear-gradient(right, transparent, #fff 55%);
    background: -moz-linear-gradient(right, transparent, #fff 55%);
    background: linear-gradient(to right, transparent, #fff 55%);
}
```

注：

>
1、将height设置为line-height的整数倍，防止超出的文字露出。
2、给p::after添加渐变背景可避免文字只显示一半。
3、由于ie6-7不显示content内容，所以要添加标签兼容ie6-7（如：<span>…<span/>）；兼容ie8需要将::after替换成:after。
