---
layout: post
keywords: css 布局
description: css会用到的各种布局
title: "css layout"
tags: [css]
date: 2018-03-18
categories: css
cover: 'https://binarycaptain.github.io/assets/img/web-cg-3.jpg'
tags: css
subtitle: 'css'

---


## css布局的发展

在Web布局整个演进过程当中，经历了没有任何布局、表格布局、定位布局、浮动布局、Flexbox布局等布局模式。除了这些我们常看到的布局之外，即将还会有Grid、Shapes（类似杂志不规则布局）这些现代的布局模式。这些布局模式从侧面也反映出其自身是Web演进过程中的一种产物，都承载了自己在当时那个时期的史命。

对于布局，我们还可以按类型和功能来进行分类。先来看按类型分类的布局模式：

* 无任何布局模式
* 表格布布局模式
* 浮动布局模式
* 定位布局模式
* 多列布局模式
* Flexbox布局模式
* Grid布局模式
* 不规则布局模式

另外除了类型分类，还可以按功能模式来分类：

* 静态布局（Static Layout）
* 流式布局（Liquid Layout）
* 自适应布局（Adaptive Layout）
* 响应式布局（Responsive Layout）

## 无任何布局模式

历史上最早的一个网页是1990年12月20日，欧洲核子研究组织（CERN）的科学家家蒂姆.伯纳斯.李在瑞士的研究中心创建的，最初仅为CERN内部的科学家所使用。在这个阶段，网站的内容主要是文字内容和图片为主，制作方法非常的简易。比如万维网（WWW），欧洲核子研究组织的一帮科学家为了方便看文档，传论文而创造的。这个时候的Web网页主要是基于Document。Document就是用标记语言加上超链接写成的由文字和图片构成的HTML页面，这样的功能已经完全能满足学术交流的需要，所以网页的早期形态和Document一样，完全基于HTML页面，并且所有内容都是静态的。比如下图所示：
![](https://binarycaptain.github.io/assets/img/css-layout-1.png)

## 表格布局模式

很长的一段时间，多页就只有文字信息和图片组成，而且只是为了查看内容。为了让页面好看一点，主要装饰还是依赖于HTML自带的标签的属性。随着Web技术的发展，HTML也越来越成熟。而Web呈现给用户的不仅仅是文字浏览，网络制作者希望制作出来的页面能更适合用户的浏览，或者想让其能像Word这样的排版软件制作出来的文档。这个时候很多网页开始依赖HTML的表格标签来对Web进行布局。在那个时候喜欢使用可视化软件来直接制作网页，比如FrontPage、DW这样的。下图就是DW制作网页的一个简易图：

也可以说表格的布局是Web早期CSS不存在的时候兴起的，是对table标签不正规使用（它是表格标签），天生就是用来显示数据的，而不是用来给网页布局的。

W3C说，表格可以用来容纳文字、图片、链接、表单以及表格等。但表格不应该单纯用来做网页布局，理由是，当Web被非可视化设备渲染的时候，表格会出问题，他们指定是屏幕阅读器以及盲文浏览器。另外，表格在大型显示设备上会强迫用户左右滚动。虽然W3C说不建议用表格来布局，特别是在CSS出现之后，更是被人嫌弃，事实上对表格的责难主要有：

* 代码臃肿
* 页面渲染性能问题
* 不利于搜索引擎优化
* 可访问性差
* 不够语义
事实上，表格也并不是一无是处，他也有自己的优点：

可观性好，当用户插入一个表格的时候就可以立即看到效果
简单方便，适合初入门的用户操作
可读性好，稍微懂一点HTML的同学就能看懂
特别是CSS越来越成熟的时候，表格对于布局而言已经退出了历史的舞台，但并不是说<table>标签就再也不使用了。前面也说过了，<table>表生就是用来显示数据的，所以不管什么时候，表格用来显示数据还是最佳的一种方式。

如果你追求完美，又不想使用表格的显示数据，那也不要紧，你可以使用table和table-cell来模拟表格的布局模式。比如：

```HTML
<table>                     //display:table;
    <thead>                 //display:table-header-group;
        <tr>                //display:table-row;
            <td>1</td>      //display:table-cell;
            <td>2</td>
        <tr>
    </thead>
    <tbody>                 //display:table-row-group;
        <tr>
            <td>3</td>
            <td>4</td>
        <tr>
    </tbody>
</table>  
```

特别是在移动端，如果你想对底部的工具栏有一个等分列的布局效果，使用display:table和display:table-cell将也是种完美的方案。只不过你需要添加width: 100%和table-layout:fixed。

## 定位布局

随着CSS的强大和HTML更多元素标签的出现，布局不在局限于表格。而对于后端或者说早期的前端人员（网页设计师），常采用的布局是CSS中的position各属性（fixed、absolute等）来布局。这种方式的布局能让你快速达到想要的布局效果。当然也有很多同学直接尝试采用 PSD2HTML这样的类似工具，直接将设计图转换成Web页面。虽然这种方式能快速实现Web的布局效果，但也受到很多的局限性：

* 需要明确指定元素的大小
* 需要明确计算元素位置坐标
* 难于维护

或许其中还有很多其他不利的因素。因此，这样的布局也并算是一种好的布局模式，但对于不太懂CSS的同学而言，这是一种简单易懂的布局。

在实际布局当中，模态弹出框和固定页头，页脚有常见定位布局的身影，特别是水平垂直居中：

```css
.center{
    width:300px;
    position:absolute;
    top:50%;
    left:50%;
    transform:translate(-50%,-50%);
}
无需知道元素大小，如果知道元素大小则可以使用margin负值来进行调整。
```

如果你使用PSD2HTML这样的工具转换出来的页面（特别早期的Photoshop或者Firework制图软件切片导出的页面），基本上都是使用定位布局。

## 浮动布局

其实CSS中的浮动布局，它的初衷并不是用来布局的，而是用来处理文本的一种排版方式。但是广大的CSSer发挥其无穷的智慧，硬是将其用于Web的布局中，而且这种布局方式成为一种主流的布局方式，并且持续了很多年。直到Flexbox布局的出现和移动端的兴起，浮动布局才慢慢的被其取替。

特别是在2004年傅捷、王宗义和祝军翻译了美国塞尔达曼（Zeldman J.）的著作《网站重构》一书。

![](https://binarycaptain.github.io/assets/img/web-cg-3.jpg)

这本书受到广大Web爱好者的青眯，可以说让国内整个前端行业（那时候还没有前端这样的职位）发生了很大的一个变化。我记得那时候，淘宝UED说：“我们要做地球上最优秀的前端”。

这本书称得上是给整个行业带来了革命性的变化，而就这场革命也造就了“21世界最大的IT冤案”。为什么说是21世界最大的IT冤案呢？只要2004年以后看了这本书的同学（并不是所有同学(^_^)），只要看到Web页面源码中有table标签，就会说这个不行，写这个页面的人不专业，页面也是垃圾，不符合W3C规范。其实这本书从来也没有说网页出现table标签就是垃圾网页，就是不符合W3C标准的页面。

除了造成21世纪最大的IT冤案之外，还有灾难性的DIV+CSS的泛滥。出现最多的词就是div，大家觉得我会div，我就很高大上。而且整个页面下来，除了div，就是div。什么p标签、span标签基本上是找不到。这个时候就是div的泛滥，根本也没有什么语义化，可读性一说。

感觉有点跑题了，还是回到正题中来。在使用浮动布局这些年当中，从中还演变出来很多布局方式，比如后面要聊的静态布局、流式布局、自适应布局和响应式布局等。

在CSS的布局模式当中，浮动布局经历的时期是最长的，持续了十多年的历史。在这个时期也演变出很多经典的布局。其中要属“圣杯”和“双飞翼”两者最为经典。这两种方法实现的都是三栏布局，两边的固定宽度，中间自适应，它们实现的效果是一样的，差别只是实现的思想。

## 圣杯布局

圣杯布局：左、中、右。中间的宽度为100%，独占一行。使用负边距（margin-left）把左右两列拉到和中间列同一行。

* 左列使用margin-left:-100%
* 右列使用margin-left: -右列宽度

同时左、中、右三列的容器设置左右padding来给左右两列留下相应宽度（左、右列宽度）。

```html
<!-- 圣杯的 HTML 结构 -->
<div class="container">
    <!-- 中间的 div 必须写在最前面 -->
    <div class="middle">中间弹性区</div>
    <div class="left">左边栏</div>
    <div class="right">右边栏</div>
</div>

/*CSS*/
.container {
    width: 480px;
    margin: 20px auto;
    padding-left: 240px;
    padding-right: 240px;
    overflow: hidden;
    border: 1px solid red;
}

.middle {
    width: 100%;
    background: green;
    float: left;
}

.left {
    margin-left: -100%;（相对于父元素，向右移动整个middle距离）
    width: 220px;
    background: orange;
    position: relative;
    float: left;
    left: -240px;
}

.right {
    margin-left: -220px（相对于右边移动整个left距离，右边的右边）;
    width: 220px;
    background: yellow;
    position: relative;
    right: -240px;
    float: left;
}
```

## 双飞翼布局

双飞翼布局和圣杯布局类似，也是左，中，右三列，中列里面会再套一个容器。

* 中列宽度设置为100%
* 使用负边距margin-left把左右两列拉到和中列同一行
* 在中列内的容器div设置margin-left和margin-right给左右两列留下对应的空间

实现代码也很简单：

```html
<!-- 双飞翼的 HTML 结构 -->
<div class="container">
    <!-- 中间的 div 必须写在最前面 -->
    <div class="middle">
        <div class="middle-inner">中间弹性区</div>
    </div>
    <div class="left">左边栏</div>
    <div class="right">右边栏</div>
</div>

/*CSS*/
.container {
    width: 960px;
    margin: 20px auto;
    overflow: hidden;
}
.middle {
    float: left;
    width: 100%;
}
.middle-inner {
    margin: 0 240px; /*留出距离*/
    background-color: yellow;
}
.left {
    float: left;
    width: 220px;
    margin-left: -100%;（相对于父元素的宽度）
    background-color: red;
}
.right {
    float: left;
    width: 220px;
    margin-left: -220px;
    background-color: green;
}
```

圣杯布局和双飞翼布局解决的问题是一样的，都是两边定宽，中间自适应的三栏布局，都是采用margin负值，将2边内容拉倒一行。中间栏要在放在文档流前面以优先渲染。这样做主要是因为早年的网络和设备没有现在这么优秀，为了让主要的内容先向用户呈现，所以很多时候都使用这两种布局方式。甚至可以说，现在很多人都还在使用这两种布局方式。但如果你继续阅读后面的内容之后，你会慢慢放弃浮动布局。虽然在布局的历史中，他承载了相当长的一段时间的使命，但有更好的，更适合的方式，我们应该要学会选择与放弃。

## 多列布局

这里要说的多列布局，并不是前面的布局模式产生的多列模式。而是CSS的Multi-column布局模块。你可能想得到，这个模块给予了我们脱离position和float这些属性，就能在网页上实现多列布局的能力。

同样，根据容器的大小，就可以控制创建栏目的数量，这是非常了不起的一个特点。Multi-column对应的CSS属性还具有以下一些功能：

定义栏目的最大宽度
定义在多栏目之间的间距
在多个栏目中平均分配好显示的内容
Multi-column好就好在能够自动为你安排好流体内容，你用不着计算确定栏目的数量，让他们排排站好就行了。

这种布局，就算是在现在，使用的人也不多，必须像报纸这样的多列布局的风格并不多，但借助他的特性，我们可以实现一些特殊的[布局风格](https://www.w3cplus.com/css/pure-css-create-masonry-layout.html)。

## Flexbox布局

flexbox布局可以具体参见阮一峰老师的flex布局。

## 网格布局

这部分我自己缺少实践，日后再补上。