---
layout: post
keywords: js 倒计时
description: js实现活动精确倒计时
title: "倒计时"
tags: [js 倒计时]
date: 2018-07-26
categories: js
cover: 'https://binarycaptain.github.io/assets/img/countdown.png'
tags: js
subtitle: 'js倒计时'
---

## 坑点

前端页面倒计时功能在很多场景中会用到，如运营活动开始倒计时和活动结束倒计时，又如购物网站的秒杀倒计时，抢购倒计时，最近的话费代付项目中，也涉及倒计时功能，但在开发过程中遇到一些麻烦和坑点，下面和大家分享一下最后是如何解决的。

在最近的倒计时活动中，就有产品吐槽两个手机在不同时间点打开同一个活动显示的开抢倒计时不一样，误差大的甚至相差几分钟，导致某些用户在活动显示还未开始红包就已被抢完了。为什么误差会这么大呢？

对于这个问题，前台开发同学一般会猜测是这个原因：倒计时读取了客户端时间造成的，因为客户端时间和服务端时间有误差，应该读取服务器时间。

是的，倒计时不应该读取客户端时间，客户端时间用户可以随时调整，会造成不一致，应读取服务器返回时间。但实践证明，做了这一步还未够，页面运行时间长了，新开的页面和原打开页面还是存在误差。

京东团购也存在这个问题：

![](https://binarycaptain.github.io/assets/img/countdown.png)

造成误差的原因主要有几种可能：

1. 没有考虑js冻结运行耗费时间；（特别是移动端容易出现，下滑页面时倒计时不动了）

2. 没有考虑页面渲染和函数运行累积时间；（京东的误差貌似属于这种）

3. 其他代码逻辑问题（这种情况就复杂了，这里不讨论）；

## 计时器原理

倒计时功能离不开setTimeout或setInterval这两个函数，要用好这两个函数必先了解好Javascript解释器的工作原理，前端界大牛John.Resig (jQuery作者) 有篇文章很好讲解了Javascript解释器工作原理 — 《How JavaScript Timers Work》。

>前端开发同学都知道，javascript是单线程的(web worker除外），更好理解的解释是javascript解释器是单线程工作，它不能在处理一个ajax的callback的同时去处理click event的callback，而是必须按照先后队列顺序执行。

![](https://binarycaptain.github.io/assets/img/countdown-1.png)

这图包含信息量很大，这里按照自己的理解描述一下：

这图从上往下看，垂直方向是时间，以ms为单位，蓝色模块是执行代码所占的时间段，如第一个代码模块执行js占用了约18ms, 第二个模块执行js占用了约11ms，其他模块类似。由于js是单线程执行，同一时间只能执行一个js代码（同一时间其他异步事件执行会被阻塞 ) , 当异步事件发生时，它会进入代码执行队列，执行线程空闲时依照队列顺序依次执行代码。

第一个模块初始化了两个定时器，一个10ms延迟的setTimeout和10ms的setInterval。这些定时器可能会在我们第一个代码块执行结束之前就触发，这取决于定时器在第一个代码块中启动的位置和时间。注意，定时器虽然触发了，但是并不会立即执行，它只是把需要延迟执行的函数按时间先后加入了执行队列，在线程的某一个空闲的时间点，这个函数就能够得到执行。

按照第一个模块事件触发的顺序（Mouse Click Occurs -. 10ms Timer Fires），第一个模块代码执行结束后，按照队列中等待的先后顺序执行事件，先执行Mouse Click CallBack再执行Timer。在执行Mouse Click CallBack模块时，Interval第一次触发未执行加入队列。在执行Timer模块时，Interval第二次触发未执行加入队列。待Mouse Click CallBack和Timer模块都执行完毕后，再依次执行队列中已触发的Interval事件。后面模块由于没有阻塞的事件了，所以按照既定10ms执行Interval事件。

## 倒计时问题

如果上面Javascript计时器原理理解了，就很好明白倒计时功能存在问题的隐患。

先看一段测试代码：

```javascript
var start = new Date().getTime();
var count = 0;

//定时器测试
setInterval(function() {
    count++;
    console.log(new Date().getTime() - (start + count * 1000));
    },1000);

```
目测代码就知道运行结果，定时器每秒执行一次，每次输出应该是0 。

实际输出：

![](https://binarycaptain.github.io/assets/img/countdown-2.png)

结论：由于代码执行占用时间和其他事件阻塞原因，导致有些事件执行延迟了几ms，但影响很微。

下面加一段阻塞代码看看：

```javascript
var start = new Date().getTime(); 
var count = 0; 
 
//占用线程事件 
setInterval(function(){ 
     var j = 0; 
     while(j++ < 100000000); 
}, 0); 
 
//定时器测试
setInterval(function(){ 
     count++; 
     console.log( new Date().getTime() - (start + count * 1000)); 
},1000);

```

实际输出：

![](https://binarycaptain.github.io/assets/img/countdown-3.png)

结论：由于加了很占线程的阻塞事件，导致定时器事件每次执行延迟越来越严重。

由于实际项目中，执行计时器的同时，会有很多其他异步阻塞事件，会导致倒计时功能不精确。

## 解决思路

这里先分析一下从获取服务器时间到前端显示倒计时的过程：

1. 客户端http请求服务器时间；

2. 服务器响应完成；

3. 服务器通过网络传输时间数据到客户端；

4. 客户端根据活动开始时间和服务器时间差做倒计时显示；

![](https://binarycaptain.github.io/assets/img/countdown-4.png)

服务器响应完成的时间其实就是服务器时间，但经过网络传输这一步，就会产生误差了，误差大小视网络环境而异，这部分时间前端也没有什么好办法计算出来，一般是几十ms以内，大的可能有几百ms。

可以得出：当前服务器时间 = 服务器系统返回时间 + 网络传输时间 + 前端渲染时间 + 常量（可选），这里重点是说要考虑前端渲染的时间，避免不同浏览器渲染快慢差异造成明显的时间不同步，这是第一点。（网络传输时间忽略或加个常量呗）

获得服务器时间后，前端进入倒计时计算和计时器显示，这步就要考虑js代码冻结和线程阻塞造成计时器延时问题了，我的思路是通过引入计数器，判断计时器延迟执行的时间来调整，尽量让误差缩小，不同浏览器不同时间段打开页面倒计时误差可控制在1s以内。

关键实现代码如下：

```javascript
//继续线程占用
setInterval(function(){ 
     var j = 0; 
     while(j++ < 100000000); 
}, 0); 
 
//倒计时
var    interval = 1000,
       ms = 50000,  //从服务器和活动开始时间计算出的时间差，这里测试用50000ms
       count = 0,
       startTime = new Date().getTime();
if( ms >= 0){
       var timeCounter = setTimeout(countDownStart,interval);                  
}
 
function countDownStart(){
       count++;
       var offset = new Date().getTime() - (startTime + count * interval);
       var nextTime = interval - offset;
       var daytohour = 0; 
       if (nextTime < 0) { nextTime = 0 };
       ms -= interval;
       console.log("误差：" + offset + "ms，下一次执行：" + nextTime + "ms后，离活动开始还有：" + ms + "ms");
       if(ms < 0){
              clearTimeout(timeCounter);
       }else{
              timeCounter = setTimeout(countDownStart,nextTime);
       }
 }

```
![](https://binarycaptain.github.io/assets/img/countdown-5.png)

>结论：由于线程阻塞延迟问题，做了setTimeout执行时间的误差修正，,保证setTimeout执行时间一致，同时也保证了在除去误差之后的时间立刻执行，保证精确度。若冻结时间特别长的，还要做特殊处理。

























