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

#### 关于Cookie和Session

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

#### 关于iframe

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

#### 不受同源限制的情况

当然也有不受同源限制的情况存在，主要有以下列举的：

1. script标签允许跨域嵌入脚本，稍后介绍的JSONP就是利用这个“漏洞”来实现。
2. img标签、link标签、@font-face不受跨域影响。
3. video和audio嵌入的资源。
4. iframe载入的任何资源。（不是iframe之间的通信）
5. <object>、<embed>和<applet>的插件。
6. WebSocket不受同源策略的限制。

## 开发时常用解决方案

#### CORS——跨域资源共享

##### 什么是跨域资源共享？

>CORS是一个W3C标准，全称是“跨域资源共享”（Cross-origin resource sharing）。

它允许浏览器向跨源服务器发出XMLHttpRequest请求，从而克服了AJAX只能同源发送请求的限制。

实现CORS主要在于服务器的设置，关键在于服务器HTTP响应报文首部的设置。前端部分大致还是跟原来发AJAX请求没什么区别，只是需要对AJAX进行一些相关的设置，稍后我们都会讲到。

##### CORS的两种请求

在讲解如何实现跨域资源共享的时候，我们先来看一下CORS的两种请求。

浏览器将CORS分为两种请求，一种是简单请求，另外一种对应的肯定就是非简单请求。

只要同时满足下面两大条件，就属于简单请求：

请求的方法是一下的三种方法之一：

HEAD
GET
POST
HTTP的头信息不超过以下几种字段：

Accept
Accept-Language
Content-Language
Last-Event-ID
Content-Type: 只限于三个值：application/x-www-form-urlencoded、multipart/formdata、text/plain。
凡是不同时满足以上两种条件，就属于非简单请求。

浏览器对于两种请求处理是不一样的。

###### 简单请求

对于简单请求，浏览器直接发出CORS请求。具体来说，就是在HTTP请求报文首部，增加一个Origin字段。如下：

```html 
GET /cors HTTP/1.1

Origin: http://api.bob.com

Host: api.alice.com

Accept-Language: en-US

Connection: keep-alive

User-Agent: Mozilla/5.0...
```

上面Origin字段的用来说明本次请求来自哪个源（协议+域名+端口）。服务器根据这个值，决定是否同意这次请求。

如果Origin指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含Access-Control-Allow-Origin字段（详见下文），就知道出错了，从而抛出一个错误，被XMLHttpRequest的onerror回调函数捕获。注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。

如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。

```html
Access-Control-Allow-Origin: http://api.bob.com

Access-Control-Allow-Credentials: true

Access-Control-Expose-Headers: FooBar

Content-Type: text/html; charset=utf-8
```
上面的HTTP响应报文首部信息中，有三个与CORS请求相关的字段，都是以Access-Control-开头。

**Access-Control-Allow-Origin**
该字段是必须的，它的值要么是请求Origin字段，要么是一个 * ，表示接受任意域名的请求。

**Access-Control-Allow-Credentials**
该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。设为true，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可。

值得一提的是，如果想要CORS支持Cookie，不仅要在服务器指定HTTP响应报文首部字段，还需要在AJAX中打开withCredentials的属性。（jQuery中AJAX设置后面会讲到）

```javascript
    var xhr = new XMLHttpRequest();
    xhr.withCredentials = true;
```

有些浏览器在省略withCredentials设置的时候，还是会发送Cookie。于是，可以显式关闭这个属性。

```javascript
    xhr.withCredentials = false;
```

需要注意的是，如果要发送Cookie，Acess-Control-Allow-Origin不能设置为 * ，必须设置成具体的域名，如果是本地调试的话可以考虑设置成null。

**Access-Control-Expose-Headers**
该字段可选。CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定。上面的例子指定，getResponseHeader('FooBar')可以返回FooBar字段的值。

###### 非简单请求

非简单请求是那种对服务器有特殊要求的请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json。

非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

下面是一段JavaScript脚本：

```javascript
    var url = 'http://api.alice.com/cors';
    var xhr = new XMLHttpRequest();
    xhr.open('PUT', url, true);
    xhr.setRequestHeader('X-Custom-Header', 'value');
    xhr.send();
```

很明显，这是一个非简单请求，使用了PUT方法来发送请求，并且自定义了一个HTTP请求报文的首部字段。

于是，浏览器发现这是一个非简单的请求，就自动发出了一个“预检”请求，要求服务器确认可以这样请求。下面是这个“预检”请求的HTTP头信息。

```html 
OPTIONS /cors HTTP/1.1

Origin: http://api.bob.com

Access-Control-Request-Method: PUT

Access-Control-Request-Headers: X-Custom-Header

Host: api.alice.com

Accept-Language: en-US

Connection: keep-alive

User-Agent: Mozilla/5.0...
```
"预检"请求用的请求方法是OPTIONS，表示这个请求是用来询问的。头信息里面，关键字段是Origin，表示请求来自哪个源。

除了Origin字段，“预检”请求的头信息还包括两个特殊字段。

**Access-Control-Request-Method**

该字段是必须的，用来列出浏览器的CORS会用到哪些HTTP方法，上面是PUT。

**Access-Control-Request-Headers**

该字段是一个用逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息。上面的例子是X-Custom-Header。

于是，服务器收到“预检”请求之后，检查了Origin、Access-Control-Request-Method和Access-Control-Request-Headers字段以后，确认允许跨域请求，就可以做出回应。

```html
HTTP/1.1 200 OK

Date: Mon, 01 Dec 2008 01:15:39 GMT

Server: Apache/2.0.61 (Unix)

Access-Control-Allow-Origin: http://api.bob.com

Access-Control-Allow-Methods: GET, POST, PUT

Access-Control-Allow-Headers: X-Custom-Header

Content-Type: text/html; charset=utf-8

Content-Encoding: gzip

Content-Length: 0

Keep-Alive: timeout=2, max=100

Connection: Keep-Alive

Content-Type: text/plain
```
如果浏览器否定了"预检"请求，会返回一个正常的HTTP回应，但是没有任何CORS相关的头信息字段。这时，浏览器就会认定，服务器不同意预检请求，因此触发一个错误，被XMLHttpRequest对象的onerror回调函数捕获。控制台会打印出如下的报错信息。

```html
XMLHttpRequest cannot load http://api.alice.com.

Origin http://api.bob.com is not allowed by Access-Control-Allow-Origin.

服务器回应的其他CORS相关字段如下：

>Access-Control-Allow-Methods: GET, POST, PUT

Access-Control-Allow-Headers: X-Custom-Header

Access-Control-Allow-Credentials: true

Access-Control-Max-Age: 1728000

```
对比简单请求服务器响应的CORS字段，发现多了三个：

**Access-Control-Allow-Methods**

该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。

**Access-Control-Allow-Headers**

如果浏览器请求包括Access-Control-Request-Headers字段，则Access-Control-Allow-Headers字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。

**Access-Control-Max-Age**
该字段可选，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期是20天（1728000秒），即允许缓存该条回应1728000秒（即20天），在此期间，不用发出另一条预检请求。

于是，一旦浏览器通过了“预检”，以后每次浏览器正常的CORS请求，都跟简单请求一样，会有一个Origin头信息字段。服务器的回应，也都有一个Access-Control-Allow-Origin头信息字段。如果开启了Cookie设置，那还有一个Access-Control-Allow-Credentials:true。

##### 如何在Node实现跨域资源共享？

那怎么在Node中结合Express设置后台的跨域部分呢？

其实很简单，需要设置的就是上面所述的几个响应首部的字段，主要考虑两种类型的请求和是否需要使用Cookie。具体设置如下：

```html
	app.all(" * ", function(req, res, next) { 

    res.header("Access-Control-Allow-Origin", /* url | * | null */);

    res.header("Access-Control-Allow-Headers", "Authorization, X-Requested-With");

    res.header("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS"); /* 服务器支持的所有字段 */

    res.header("Access-Control-Allow-Credentials", "true"); /* 当使用Cookie时 */

    res.header("Access-Control-Max-Age", 300000); /* 设置预检请求的有效期 */

    if (req.method === "OPTIONS") return res.send(200); /*让options请求快速返回*/

    else next();
});
```
上面的设置有几个需要注意的地方：

如果需要本地调试，也就是在本地HTML页面发请求（类似file://...之类的url），可以把Access-Control-Allwo-Origin的值设置为Null，这样子就能够使用Cookie。如果设置成 * ，虽然也可以跨域发送请求，但是这个时候没有办法使用Cookie。
Access-Control-Allow-Headers字段不是必须的，仅当发送的请求有Access-Control-Request-Headers时需要设置。
使用Cookie的时候要配置Access-Control-Allow-Methods字段。
“预检”请求可设置缓存时间，可以保证不多次发送“预检”请求。
需要判断是否为“预检”请求，当拦截“预检”请求的时候直接返回。
如果使用jQuery封装的AJAX发送请求，那么需要在相应的JS代码设置：

```javascript
$.ajaxSetup({ xhrFields: { withCredentials: true }, crossDomain: true });
```
withCredentials是设置CORS发送Cookie，默认是不发送的。
crossDomain告知AJAX允许跨域。

以上就是CORS设置跨域的具体介绍。

#### 基于JSONP技术实现跨域

##### JSONP原理

对于JSONP来说，前面也已经提到了，其实它是利用了某些不受同源限制的标签的所谓“漏洞”，来实现“曲线救国”式的跨域的方案。

它借用script标签不受同源限制的这个特性，通过动态的给页面添加一个script标签，利用事先声明好的数据处理函数来获取数据。

值得一提的是，JSONP这种方法其实和CORS有很大的区别，它并不属于一种规范。所谓的JSONP是应用JSON数据的一种新方法，它只不过是被包含在函数调用中的JSON。

在JSONP中包含两部分：回调函数和数据。其中，回调函数是当响应到来时要放在当前页面被调用的函数。而数据，就是传入回调函数中的JSON字符串，也就是回调函数的参数了。下面我们简单模拟一下JSONP的通信过程。

我们来简单的模拟一下JSONP的通信过程。

```javascript
        function handleResponse(response) {
            console.log(response.data);
        }
        var script = document.createElement("script");
        script.src = "http://example.com/jsonp/getSomething?uid=123&callback=hadleResponse"
        document.body.insertBefore(script, document.body.firstChild);
        /*handleResponse({"data": "hey"})*/
```

它的过程是这样子的：

1. 当我们通过新建一个script标签请求时，后台会根据相应的参数来生成相应的JSON数据。比如说上面这个链接，传递了handleResponse给后台，然后后台根据这个参数再结合数据生成了handleResponse({"data": "hey"})。
2. 紧接着，这个返回的JSON数据其实就可以被当成一个js脚本，就是对一个函数的调用。
3. 由于我们事先已经声明了这么一个回调函数，于是当资源加载进来的时候，直接就对函数进行调用，于是数据当然就能获取到了。
4. 至此，跨域通信完成。
另外，想要实现JSONP，后台服务器也必须做相应的设置。

值得一提的是，JSONP是存在一定的局限性的：

* 只能用于GET请求
* 存在安全问题，请求代码中可能存在安全隐患
* 要确定JSONP请求是否失败并不容易

#### 一个简单的JSONP实现

下面是一个实现JSONP的库，我们来一起分析一下它的源代码。这是源码的[github地址](https://github.com/webmodules/jsonp)。

```javascript
    /**
 * Module dependencies
 */

var debug = require('debug')('jsonp');
//使用依赖

/**
 * Module exports.
 */

module.exports = jsonp;
//输出模块

/**
 * Callback index.
 */

var count = 0;
//回调函数的index值，便于取名。

/**
 * Noop function.
 */

function noop(){}
//无操作空函数，以便使用后把window[id]置空

/**
 * JSONP handler
 *
 * Options:
 *  - param {String} qs parameter (`callback`)
 *  - prefix {String} qs parameter (`__jp`)
 *  - name {String} qs parameter (`prefix` + incr)
 *  - timeout {Number} how long after a timeout error is emitted (`60000`)
 *
 * @param {String} url
 * @param {Object|Function} optional options / callback //这里的callback是取得数据后的callback，不是传给服务器的callback
 * @param {Function} optional callback
 */


function jsonp(url, opts, fn){
  if ('function' == typeof opts) {
    fn = opts;
    opts = {};
  }
  if (!opts) opts = {};

  var prefix = opts.prefix || '__jp';

  // use the callback name that was passed if one was provided.
  // otherwise generate a unique name by incrementing our counter.
  var id = opts.name || (prefix + (count++));

  var param = opts.param || 'callback';
  var timeout = null != opts.timeout ? opts.timeout : 60000;
  var enc = encodeURIComponent;
  var target = document.getElementsByTagName('script')[0] || document.head;
  var script;
  var timer;

  //一定时间内后台服务器没有返回视为超时
  if (timeout) {
    timer = setTimeout(function(){
      cleanup();
      if (fn) fn(new Error('Timeout'));
    }, timeout);
  }
  //回复原始设置、清空状态
  function cleanup(){
    if (script.parentNode) script.parentNode.removeChild(script);
    window[id] = noop;
    if (timer) clearTimeout(timer);
  }
  //取消操作
  function cancel(){
    if (window[id]) {
      cleanup();
    }
  }

  //声明函数，等待script标签加载的url引入完毕后调用
  window[id] = function(data){
    debug('jsonp got', data);
    cleanup();
    if (fn) fn(null, data);//node中约定第一个参数为err，但是这里不传，直接就置为null
  };

  // add qs component
  url += (~url.indexOf('?') ? '&' : '?') + param + '=' + enc(id);
  url = url.replace('?&', '?');

  debug('jsonp req "%s"', url);

  // create script
  script = document.createElement('script');
  script.src = url;
  target.parentNode.insertBefore(script, target);
  //引入script标签后会直接去调用声明的函数，然后函数会把script标签带有的data给传出去

  return cancel;
  //返回初始状态
}
```

#### 基于JSONP库的封装

```javascript
    /* 这个是自己定义的一个_jsonp */
/**
 * @param {String} url
 * @param {Object} data
 * @param {Object} option
 * @returns
 */
function _jsonp(url, data, option) {
  url += (url.indexOf('?') < 0 ? '?' : '&') + param(data);

  return new Promise((resolve, reject) => {
    jsonp(url, option, (err, data) => {
      if (!err) {
        resolve(data);
      } else {
        reject(err);
      }
    });
  });
  /* 这里把jsonp封装成了一个promise对象，回调函数中如果成功的话会把数据带回来然后resolve出去 */
}
//紧接着是对参数的一个序列化
function param(data) {
  let url = '';
  for (var k in data) {
    let value = data[k] !== undefined ? data[k] : '';
    url += `&${k}=${encodeURIComponent(value)}`;
  }
  return url ? url.substring(1) : '';/* 这里的substring保证不会有多余的& */
}
```
#### 在jQuery中使用JSONP
另外，在jQuery中的AJAX中，已经封装了JSONP，下面简单介绍一下如何去使用。

```javascript
    $.ajax({
    type: "get",
    url: "http://example.com",
    dataType: "jsonp",
    jsonp: "callback",
    jsonpCallback: "responseCallback",
    success: function (data) {
        console.log(data);
    },
    error: function (data) {
        console.log(data);
    }
});
```
在AJAX中，主要设置dataType类型为jsonp。对于jsonp参数来说，默认值是callback，而jsonpCallback参数的值默认是jQuery自己生成的。如果想自己指定一个回调函数，可像代码中对jsonpCallback进行设置。上面的代码中，最终的url将会是http://example.com?callback=responseCallback。

## 使用代理服务器转发请求

由于同源策略仅存在于浏览器。对于服务器与服务器之间的通讯，是不存在任何同源限制的说法的。
因此，使用代理服务器来转发请求也是我们在日常开发中解决跨域的一个常用的手段。
实现的方法很简单，只要你会使用Node和Express。
需要注意的是，通常后台服务器都会自己的一个验证的机制，比如说微信文章在iframe中图片是加载不出来的，因为其后台对referer进行了验证。另外，有些服务器也会通过发送一些uid等等之类的字符串供后台校验。因此，我们在使用代理服务器的时候，要重点关注请求的参数，这样才能准确的模拟出请求并转发。

下面简单介绍如何使用代理服务器转发请求。

分析url请求所需要的参数。
代理服务器暴露出一个api，这个路由实际的功能是去请求真正的服务器。
之后，我们只要请求这个api，我们所建的代理服务器就会默认的帮我们去转发请求到真正的服务器上，其中会加上一些相应的参数。
最后，我们来利用反微信图片防盗链这个实例来写一个代理服务器。

##### 如何使用代理服务器反微信图片防盗链？
当我们上线了一个网站的时候，然后img标签引用了微信图片的地址，会出现下面的这种情况。

![](https://binarycaptain.github.io/assets/img/crossdomain-1.png)

这就是所谓的防盗链。

现在我们给它加上一个代理，代码如下：

```javascript
var express = require("express");
var superagent = require("superagent");
var app = express();

app.use("/static", express.static("public"));

app.get("/getwxImg", (req, res) => {
    //如果单纯的去获取会出现参数丢失的情况，因为出现了两个问号
    var url = req.url.substring(req.url.indexOf("param=") + 6);
    res.writeHead(200, {
        'Content-Type': 'image/*'
    });
    superagent.get(url)
        .set('Referer', '')
        .set("User-Agent",
        'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36'
        )
        .end(function (err, result) {
            if (err) {
                return false;
            }
            res.end(result.body);
            return;
        });
});

app.listen(4001, (err) => {
    if (err) {
        console.log(err);
    } else {
        console.log("server run!");
    }
});
```

><!-- 这是有代理的情况 -->
<img src="http://localhost:4001/getwxImg?param=http://mmbiz.qpic.cn/mmbiz/CoJreiaicGKekEsuheJJ7Xh53AFe1BJKibyaQzsFiaxfHHdYibsHzfnicbcsj6yBmtYoJXxia9tFufsPxyn48UxiaccaAA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&tp=webp">
<!-- 下面是没有代理的情况 -->
<img src="http://mmbiz.qpic.cn/mmbiz/CoJreiaicGKekEsuheJJ7Xh53AFe1BJKibyaQzsFiaxfHHdYibsHzfnicbcsj6yBmtYoJXxia9tFufsPxyn48UxiaccaAA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&tp=webp">

结果如下：

![](https://binarycaptain.github.io/assets/img/crossdomain-2.png)

结果显而易见，这就是所谓的代理服务器，附上[gitub项目地址](https://github.com/Devinnn/weixinPictureReverseUrl)。













