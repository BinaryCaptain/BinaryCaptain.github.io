---
layout: post
keywords: blog
description: blog
title: "跨域解决方案"
tags: 跨域 jsonp
date: 2017-05-07
categories: 跨域
cover: 'https://binarycaptain.github.io/assets/img/crossdomain.png'
subtitle: '跨域你知道吗'

---


## 浏览器同源策略

#### 什么是浏览器同源策略？

“同源策略”（Same Origin Policy）是浏览器安全的基础。

同源策略限制从一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的关键的安全机制。

在判断两个页面的url是否具有相同的源之前，我们先来看一下一个url（统一资源定位符）的基本组成部分。

对于一个url，它的基本组成部分是：协议 :// 域名（或者是ip） : 端口（如果指定了） / 路径。

那么下面我们来举个例子：

对于http://www.example.com/static来说，协议就是http，它的域名是www.example.com，它的路径是static，这里的端口号省略了（默认为80）。

值得一提的是，在这个域名下面，example.com是主域名，而www.example.com是子域名，同样的a.example.com也是一个与前面所述不同的另一个子域名。并且，上面这三个域名互不相同。

这里不再展开赘述，相关知识请查阅与计算机网络有关的知识。

那么如何判断是否同源呢？

判断同源的三要素是：协议、端口、域名。

## 关于Cookie和Session

说到同源策略，必不可少的就是Cookie这个东西了。

而讲到Cookie，跟它关联在一起的又有Session。

这里简要的总结一下：

Cookie受到浏览器同源策略的限制，A页面的Cookie无法被B页面的Cookie访问和操作。
Cookie最大存储容量一般为4KB，由服务器生成，在HTTP报文首部设置Set-Cookie可以指定生成Cookie的内容和生命周期，如果由浏览器生成则在关掉浏览器后失效。
Cookie存储在浏览器端，由于每次发送HTTP请求默认会把Cookie附到HTTP首部上去，所以Cookie主要用来身份认证，而不用来存储其他信息，防止HTTP报文过大。
Session存储在服务器，主要与Cookie配合使用完成身份认证和状态保持的功能。只有Cookie或只有Session都无法完成身份认证和状态保持的功能。

总结：2者存在的端不同，一个在浏览器端，而另外一个在服务器端，session更多的是去配合cookie。

最后，对Cookie和Session实现的身份认证和状态保持功能做一个举例。

假设现在有一个学生信息管理系统，此时数据库已经有学生的相关信息。（账号、密码、个人信息等等） 
然后当学生登录这个系统，通过POST请求把用户的账户密码发送到后台服务器。当后台服务器接收到这些参数的时候，会跟数据库保存的记录进行匹配。 
一旦匹配成功，也就是用户的账号密码都正确的情况下，这个时候后台服务器会在Session中记录一个值，可以是用户名或者其他能够唯一标识用户的字段。 
当把这个值保存在Session中后，后台服务器会返回响应告知客户端登录成功，可以进行后续的操作。此时，后台服务器会在HTTP响应报文中添加一个字段Set-Cookie，它的值是当前Session的SessionID，（这个SessionID是指向我们当前的那个Session的，在Node的Express中express-session会封装好这个过程）当然还会设置Cookie的其他属性，比如说过期时间Expires等等。

当浏览器接收到这个HTTP响应报文的时候，就会在本地设置一个Cookie，它的过期时间(一般为15天左右）由响应报文中Set-Cookie中的Expires字段的值决定，如果为空，则关闭浏览器（即会话结束时）后失效。

之后，每次向后台服务器发送请求的时候，浏览器默认会把这个Cookie加在HTTP请求报文的Cookie中。这样，每次后台服务器接收到请求的时候，会根据Cookie中的SessionID去找到我们的Session。
假如这个SessionID映射得到Session，那么这个时候说明浏览器是已经登录过了。于是，就可以进行后续的一些相关的操作。 
另外，值得一提的是，Session机制决定了当前客户只会获取到自己的Session，而不会获取到别人的Session。各客户的Session也彼此独立，互不可见。也就是说，当多个客户端执行程序时，服务器会保存多个客户端的Session。获取Session的时候也不需要声明获取谁的Session。

这就是Cookie和Session做状态保持和身份验证的一个具体的例子。

## 关于iframe

说到同源限制，还有一个不得不提的就是iframe。

iframe可以在父页面中嵌入一个子页面，在日常开发中一旦使用，避免不了的就要涉及到不同的iframe页面进行通信的问题，可能是获得其他iframe的DOM，或者是获取其他iframe上的全局变量或方法等等。

同源下的iframe，也就是iframe中的src属性的URL符合同源的条件，那么通过iframe的contentDocument和contentWindow获取其他iframe的DOM或者全局变量、方法都是很简单的事情。

那如果是非同源的两个iframe，单纯的通过变量访问的方式就受到同源限制了。

为了解决这个问题，HTML5引入了一个新的API：postMessage，主要就是用来解决存在跨域问题的iframe页面之间通信的问题。

下面简单的举一个例子，假如现在有两个不同的页面，A页面的url是http://localhost:4002/parent.html，B页面的url的是http://localhost:4003/child.html，现在我把B页面用iframe嵌在A页面下面，代码（精简）是这样子的。现在我要实现的是向子页面B传递一个消息：

A页面代码：

```html
	<body>
    <h1>A页面</h1>
    <iframe src="http://localhost:4003/child.html" id="child">
    </iframe>
    <script>
        window.onload = function() {
            document.getElementById("child").contentWindow.postMessage("父页面发来贺电", "http://localhost:4003");
        }
    </script>
</body>
```

B页面代码：

```html
<body>
    <h1>B页面</h1>
    <script>
        window.onload = function() {
            window.addEventListener("message", function(e) {
                //判断信息的来源是否来自于父页面，保证信息源的安全
                if(e.source != window.parent) return;
                alert(e.data);
            });
        };
    </script>
</body>
```

结果如图：

![图片描述](https://binarycaptain.github.io/assets/img/crossdomain.png)

postMessage接受两个参数，一个是要传送的data，另外一个是目标窗口的源，如果想传给任何窗口，可以设置成 * 。 
目标页面接收信息的时候，使用的是window.addEventListener("message", function() {})

## 残酷的北漂生活


来之前我以为北京其实没有想象中那么不好，当自己亲身体会之后才知道，北京是一个很冷酷的城市。第一个是租房问题，刚来的时候幸亏自己有亲戚在这里，然后可以慢慢找合适的房子，并不是每个人都能这样，对于很多孤身一个人来北京的人，找房子真的是一件非常累的事情，我有一个同学光找房子就花了将近半个月的时间。你心仪的房子，房租太贵，房租便宜的，你又看不上，说到底还是穷。第二个是上班问题，每天上班就要花快2个小时，刚开始真的痛苦，各种不适应，后来都麻木了，你会发现北京跟你想象中的真的差距很大，这个时候很多人真的会失去斗志，抱着混一天算一天的心态在北京呆着，这种无异于温水煮青蛙，很多北漂现在就是这种状态，这个时候一定要调整自己，别人帮不了你，只有你自己改变，好在我现在从当时那个状态中走了出来。第三是你会发现北京的人都很冷漠，也不是说他们想这样，是整个大环境逼着你不得不这么做，大家都很忙，根本没有人问你，这一点主要是我丢手机的时候深有体会，丢手机这件事情会单独写一篇文章。第四是工作，我实习的岗位是微博商务增值部的前端工程师，我实习工资是每天200，每个月租房花掉将近一半，再加上坐地铁吃饭乱七八糟的，根本剩不了多少钱。这个时候明白离开父母的庇护，真正自己一个人出来生活的艰辛，是啊，人总是要长大的。北漂的艰辛，只有自己走过，才会明白。


## 我与微博

本来我以为刚开始实习的时候，主管并不会将很困难的东西交给你去做，没想到新浪实习生不打杂，各种项目上来就让你做，公司用的是react技术栈，说实话，我之前只是了解原理，基本没有用过，一上来就写实际项目，真的是很难。好在前端leader人很好，一直也耐心教我，然后自己利用在地铁上公交上的碎片时间多看，慢慢也就熟悉了，现在自己基本可以跟着项目走了，慢慢开始接需求，工作也算是走上了正轨。其实很多时候，我一直问自己为什么要来北京，压力这么大，想了挺长时间，原因无非2个，第一个是赚钱，第二个就是实现自己的价值，赚钱没什么说的，父母年龄越来越大，不可能一直靠父母，上次回家看到我爸和我妈头上白头发都很多，其实自己心里还是很难受的，毕竟这些年父母供我读书，很费心。自己一直有一个想法，赚一些钱之后，带父母出去旅游一次，这个计划一定会执行，有时间一定要多陪陪父母，父母是唯一一直支持你的人。第二个就是实现人生价值，其实说的有点大，就是给自己找点事情做，新浪微博是一个日活量在2亿左右的APP，每天并发量非常大，对各个技术部门都是一种考验，每天面对那么多用户，那么如何保证用户体验，是每一个技术人员应该思考的问题，尤其我是做前端开发，用户体验非常重要，之前在学校做项目的时候，差几个像素，有的浏览器不兼容了这些问题根本不用考虑。微博对前端的要求是非常高的，页面还原度必须在98%以上，也就是最多差1到2个像素，设计会拿psd和你的网页一点一点对，刚开始因为这个事情真的被骂惨了。后来leader说了这么一句话，这么多人用你的产品，你做成这样，你觉得能交差吗，后来自己也严格要求自己，那段时间眼镜真的受不了，每天就看着电脑对像素，生怕出错。来微博这么久，参加了微博之夜活动，让我觉得微博真的没有白来，微博之夜当天看直播的在线的人数峰值达到了一个亿，你可以想想，中国13亿人口，有1/13的人口在用你做的产品，其实想想就很自豪，这种工作机会并不是每个人都有，也不是花钱能买到经历，虽然我只是个打酱油的角色吧，但是还是从中学习到了非常多的东西。打个广告，由我负责的微博发现页的尤物志即将上线，希望看到这篇文章的童鞋多支持。

![图片描述](https://binarycaptain.github.io/assets/img/weibo.jpeg)

![图片描述](https://binarycaptain.github.io/assets/img/youwuzhi.jpeg)

## 未来的小目标

每一个做技术的程序员都有一个去阿里和腾讯的梦想，阿里和腾讯有全中国最优秀的程序员，是每个技术人的天堂，我也不例外，阿里和腾讯或许会迟到，但一定不会缺席，2018一定尽自己最大的努力，让自己在技术上更上一层楼，争取能够加入到其中一家公司。

## 勉励

其实自己一个人在北京真的很孤单，尤其是下班回到家之后，没有人和你说话，我一开始很不习惯，回到家就把音响开到最大，生怕周围太安静，有一首我一直在听的歌与你分享


![图片描述](https://binarycaptain.github.io/assets/img/share-songs.jpeg)

Looking back, I see a setting sun
回首看见一颗下沉的夕阳

And watch my shadow fade into the floor
凝视我的影子渐渐与地面融合

I am left standing on the edge
我被独自留下伫立边缘

Wondering how we got this far
沉思我们是怎样走得如此之远

How we got this far
怎样走得如此之远

They left us alone, the kids in the dark
他们抛弃了我们，黑暗中的孩子

To burn out forever or light up a spark
要么永世燃尽，要么点亮火光

We come together, state of the art
我们聚集在一起，就像某种艺术

We'll never surrender the kids in the dark
我们永远不会投降，因为我们是黑暗中的孩子

So let the world sing
让世界呼号吧

黑暗中的孩子永远不会迷茫！！！
