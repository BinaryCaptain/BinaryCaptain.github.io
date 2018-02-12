---
layout: post
description: blog
title: 'Web缓存详解'
date: 2017-05-06
categories: 技术
cover: 'https://binarycaptain.github.io/assets/img/cache.png'
tags: 缓存 性能
subtitle: '缓存那些事儿'
---

## 缓存之于性能优化

请求更快：通过将内容缓存在本地浏览器或距离最近的缓存服务器（如CDN），在不影响网站交互的前提下可以大大加快网站加载速度。

降低服务器压力：在大量用户并发请求的情况下，服务器的性能受到限制，此时将一些静态资源放置在网络的多个节点，可以起到均衡负载的作用，降低服务器的压力。

## 缓存方式

服务端缓存,例如CDN

客户端缓存,即浏览器缓存,可通过cache,manifest等实现.

浏览器缓存分两类,强制缓存和协商缓存

强制缓存:浏览器在加载资源时，根据http header判断它是否命中强缓存，如果命中，浏览器直接从自己的缓存中读取资源，不会发请求到服务器。比如某个css文件，如果浏览器在加载它所在的网页时，这个css文件的缓存配置命中了强缓存，浏览器就直接从缓存中加载这个css，连请求都不会发送到网页所在服务器.

协商缓存：当强缓存没有命中的时候，浏览器一定会发送一个请求到服务器，通过服务器端依据资源的另外一些http header验证这个资源是否命中协商缓存，如果协商缓存命中，服务器会将这个请求返回（304），但是不会返回这个资源的数据，而是告诉客户端可以直接从缓存中加载这个资源，于是浏览器就又会从自己的缓存中去加载这个资源；若未命中请求，则将资源返回客户端，并更新本地缓存数据（200）。

强制缓存不发请求到服务器，协商缓存会发请求到服务器。

强制缓存:Expires、Cache-control

协商缓存:Last-Modified/If-Modified-Since, Etag/If-None-Match

## 强制缓存控制

Expires,HTTP/1.0提出的一个表示资源过期时间的header，它描述的是一个绝对时间，由服务器返回，用GMT格式的字符串表示，如：Expires:Thu, 31 Dec 2016 23:55:55 GMT，Expires是较老的强缓存管理header，由于它是服务器返回的一个绝对时间，这样存在一个问题，如果客户端的时间与服务器的时间相差很大（比如时钟不同步，或者跨时区），那么误差就很大，所以在HTTP/1.1版开始，使用Cache-Control: max-age=秒替代。

Cache-Control描述的是一个相对时间，在进行缓存命中的时候，利用客户端时间进行判断，所以相比较Expires，Cache-Control的缓存管理更有效.
读取缓存数据条件：上次缓存时间（客户端的）+max-age < 当前时间（客户端的）.Cache-Control取值的含义:

各个消息中的指令含义如下：

no-cache指示请求或响应消息不能缓存，该选项并不是说可以设置”不缓存“，而是需要和服务器确认。

max-age指示客户机可以接收生存期不大于指定时间（以秒为单位）的响应。上次缓存时间（客户端的）+max-age（64200s）<客户端当前时间

min-fresh指示客户机可以接收响应时间小于当前时间加上指定时间的响应。

![图片描述](https://binarycaptain.github.io/assets/img/cache.png)

## 协商缓存控制
Last-Modified/If-Modified-Since：
Last-Modified：标示这个响应资源的最后修改时间。web服务器在响应请求时，告诉浏览器资源的最后修改时间。If-Modified-Since：当资源过期时（强缓存失效），发现资源具有Last-Modified声明，则再次向web服务器请求时带上If-Modified-Since标签，把上次服务器返回的Last-Modified时间返回到服务器端。web服务器收到请求后根据If-Modified-Since 时间与被请求资源的最后修改时间进行比对。若最后修改时间较新，说明资源又被改动过，则响应整片资源内容（写在响应消息包体内），HTTP 200；若最后修改时间较旧，说明资源无新修改，则响应HTTP 304 (无需包体，节省浏览)，告知浏览器继续使用所保存的cache。

注: Last-Modified标注的最后修改只能精确到秒级，如果某些文件在1秒钟以内，被修改多次的话，它将不能准确标注文件的修改时间（无法及时更新文件）
如果某些文件会被定期生成，当有时内容并没有任何变化，但Last-Modified却改变了，导致文件没法使用缓存，有可能存在服务器没有准确获取文件修改时间，或者与代理服务器时间不一致等情形（无法使用缓存）。

Etag/If-None-Match:
Etag：web服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识（生成规则由服务器决定）。Apache中，ETag的值，默认是对文件的索引节（INode），大小（Size）和最后修改时间（MTime）进行Hash后得到的。

If-None-Match：当资源过期时（使用Cache-Control标识的max-age），发现资源具有Etage声明，则再次向web服务器请求时带上头If-None-Match （Etag的值）。web服务器收到请求后发现有头If-None-Match 则与被请求资源的相应校验串进行比对，决定返回200或304。
注:Etag是服务器自动生成或者由开发者生成的对应资源在服务器端的唯一标识符，能够更加准确的控制缓存。

上述缓存控制机制优先级:
Expires < Cache:max-age
Last-Modified < ETag

## CDN

```
CDN的全称是Content Delivery Network，即内容分发网络。通过在网络各处放置节点服务器所构成的在现有的互联网基础之上的一层智能虚拟网络，CDN系统能够实时地根据网络流量和各节点的连接、负载状况以及到用户的距离和响应时间等综合信息将用户的请求重新导向离用户最近的服务节点上。其目的是使用户可就近取得所需内容，解决 Internet网络拥挤的状况，提高用户访问网站的响应速度。
```
```
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>

</body>
<script>
    Person.prototype.constructor = Person; //函数对象的prototype属性的contructor属性指向这个函数对象
    function Parent(){
        this.surname = '张';
        this.name = '张三';
        this.like = ['apple','banana'];
    }
    var par = new Parent();
    function Child(){
        this.name = '张小三';
    }
    Parent.prototype.showSurname = function(){
        return this.surname
    }
</script>
</html>

```

CDN是一个经策略性部署的整体系统，包括分布式存储、负载均衡、网络请求的重定向和内容管理4个要件，而内容管理和全局的网络流量管理（Traffic Management）是CDN的核心所在。通过用户就近性和服务器负载的判断，CDN确保内容以一种极为高效的方式为用户的请求提供服务。