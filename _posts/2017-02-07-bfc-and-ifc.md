---
layout: post
keywords: blog
description: blog
title: "扒一下W3C规范里的BFC和IFC"
tags: [BFC IFC]
date: 2017-02-07
categories: BFC IFC
cover: 'https://binarycaptain.github.io/assets/img/bfc-1.png'
tags: BFC IFC
subtitle: 'BFC IFC'

---

先说说FC，FC的含义就是Fomatting Context。它是CSS2.1规范中的一个概念。它是页面中的一块渲染区域，并且有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素的关系和相互作用。BFC和IFC都是常见的FC。分别叫做Block Fomatting Context 和Inline Formatting Context。

## BFC
BFC（Block Formatting Context）叫做“块级格式化上下文”。BFC的布局规则如下：

1. 内部的盒子会在垂直方向，一个个地放置；
2. 盒子垂直方向的距离由margin决定，属于同一个BFC的两个相邻Box的上下margin会发生重叠(按照数值较大的margin来)；
3. 每个元素的左边，与包含的盒子的左边相接触，即使存在浮动也是如此(右边同理)；
4. BFC的区域不会与float重叠；
5. BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素，反之也如此；
6. 计算BFC的高度时，浮动元素也参与计算。


介绍过了BFC的布局规范，再来说说哪些元素会产生BFC。
1. 根元素；
2. float的属性不为none；
3. position为absolute或fixed；
4. display为inline-block，table-cell，table-caption，flex；
5. overflow不为visible

那么，我们一般如何触发BFC呢？需要触发（需要写一行声明，来告诉外界声明独立）建议使用overflow：hidden来声明，在包住的div里触发，要触发哪一个就在哪一个地方写上overfolw:hidden

## 三、BFC的作用及原理

1. 自适应两栏布局

```html
<style>
    body {
        width: 300px;
        position: relative;
    }

    .aside {
        width: 100px;
        height: 150px;
        float: left;
        background: #f66;
    }

    .main {
        height: 200px;
        background: #fcc;
    }
</style>
<body>
    <div class="aside"></div>
    <div class="main"></div>
</body>
```
![](https://binarycaptain.github.io/assets/img/bfc-1.png)

根据BFC布局规则第3条：

>每个元素的margin box的左边， 与包含块border box的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。

因此，虽然存在浮动的元素aslide，但main的左边依然会与包含块的左边相接触。

根据BFC布局规则第四条：

>BFC的区域不会与float box重叠。

我们可以通过通过触发main生成BFC， 来实现自适应两栏布局。

```css
.main {
    overflow: hidden;
}
```

　当触发main生成BFC后，这个新的BFC不会与浮动的aside重叠。因此会根据包含块的宽度，和aside的宽度，自动变窄。效果如下：

![](https://binarycaptain.github.io/assets/img/bfc-2.png)

2. 清除内部浮动
```html
<style>
    .par {
        border: 5px solid #fcc;
        width: 300px;
    }

    .child {
        border: 5px solid #f66;
        width:100px;
        height: 100px;
        float: left;
    }
</style>
<body>
    <div class="par">
        <div class="child"></div>
        <div class="child"></div>
    </div>
</body>
```
根据BFC布局规则第六条：

>计算BFC的高度时，浮动元素也参与计算

为达到清除内部浮动，我们可以触发par生成BFC，那么par在计算高度时，par内部的浮动元素child也会参与计算。

代码：

```css
.par {
    overflow: hidden;
}
```
效果如下：

![](https://binarycaptain.github.io/assets/img/bfc-3.png)

3. 防止垂直 margin 重叠

代码:
```css
<style>
    p {
        color: #f55;
        background: #fcc;
        width: 200px;
        line-height: 100px;
        text-align:center;
        margin: 100px;
    }
</style>
<body>
    <p>Haha</p>
    <p>Hehe</p>
</body>
```
页面：

![](https://binarycaptain.github.io/assets/img/bfc-4.png)

两个p之间的距离为100px，发送了margin重叠。(因为这里的根元素是body，所以形成了BFC，因此其内部的元素的margin会发生重叠)

根据BFC布局规则第二条：

>Box垂直方向的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生重叠

我们可以在p外面包裹一层容器，并触发该容器生成一个BFC。那么两个P便不属于同一个BFC，就不会发生margin重叠了。

代码：
```html
<style>
    .wrap {
        overflow: hidden;
    }
    p {
        color: #f55;
        background: #fcc;
        width: 200px;
        line-height: 100px;
        text-align:center;
        margin: 100px;
    }
</style>
<body>
    <p>Haha</p>
    <div class="wrap">
        <p>Hehe</p>
    </div>
</body>
```
效果如下:
![](https://binarycaptain.github.io/assets/img/bfc-5.png)

## 总结
其实以上的几个例子都体现了BFC布局规则第五条：

>BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。

因为BFC内部的元素和外部的元素绝对不会互相影响，因此， 当BFC外部存在浮动时，它不应该影响BFC内部Box的布局，BFC会通过变窄，而不与浮动有重叠。同样的，当BFC内部有浮动时，为了不影响外部元素的布局，BFC计算高度时会包括浮动的高度。避免margin重叠也是这样的一个道理。

## IFC
IFC(inline formatting context),即⾏内格式化上下⽂，与之对应的是BFC(block formating context),块格式化上下⽂，它和BFC⼀样，既不是属性也不是元素，⽽是⼀种环境，⼀种上下⽂。

在IFC中，框（boxes）⼀个接⼀个地⽔平排列，起点是包含块的顶部。⽔平⽅向上的margin，border和padding在框之间得到保留。框在垂直⽅向上可以以不同的⽅式对⻬：它们的顶部或底部对⻬，或根据其中⽂字的基线对⻬。包含那些框的⻓⽅形区域，会形成⼀⾏，叫做⾏框(line box)。

一个line box的宽度，由他的包含块(containg block)和floats的存在情况决定。linebox的高度，由你给出的代码决定。

行高的高度足以包含它的内部容器，也可能比它包含的容器们都高（比如在基线对齐的时候），当它包含的内部容器的高度小于行框的高度时，内部容器的垂直位置由vertical属性来确定，这个性质可以用来实现垂直居中。

当几个行内框在水平方向无法放入一个行内框时，它们可以分配在两个或多个垂直堆叠的行框中。因此，一个段落就是行框在垂直方向上的堆叠。行框在堆叠时没有垂直方向上的分割且永不重叠。

一个行内框超出包含它的行框的宽度，它将会被分割为几个框。如果一个行框不能被分割，行内框会益处行框。

如果一个行内框被分割，margin、padding、border在所有分割处没有视觉效果。

创建一个IFC的环境，让行框的高度是包含块的高度的100%,让行框内部的元素使用Vertical-align：middle,就可以实现垂直居中。











