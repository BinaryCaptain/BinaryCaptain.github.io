---
layout: post
keywords: react
description: 如何实现一个angular的$q
title: "angular"
tags: [angular]
date: 2018-09-04
categories: angular
cover: 'https://binarycaptain.github.io/assets/img/angular-4.png'
tags: angular
subtitle: '手写一个angular的$q'
---

## 前言

$q是angular中非常常用的一个API，$q中的许多方法和ES6中的promise中的很多方法十分类似，下面我们通过阅读angular$q的源码来理解Promise的实现原理和思想。Promsise的规范来源于Promises/A+社区，它有很多版本的实现。

## Pormise的典型应用
```javascript
let promise=new Promise((resolve,reject)=>{
    let flag=false //异步操作状态
    if(flag){
        resolve('成功了')
    }else{
        reject('失败了')
    }
})
promise.then(data=>{
    console.log("s",data)
},err=>{
    console.log("e",err)
})
```
可以看到上面是最简单一个Promise的案例,那么原生Promise又是如何实现的呢，下面我们就来一步一步实现一个Promise

### 构造函数

因为标准并没有指定如何构造一个Promise对象，所以我们同样以目前一般Promise实现中通用的方法来构造一个Promise对象，也是ES6原生Promise里所使用的方式，即：

```javascript

// Promise构造函数接收一个executor函数，executor函数执行完同步或异步操作后，调用它的两个参数resolve和reject
var promise = new Promise(function(resolve, reject) {
  /*
    如果操作成功，调用resolve并传入value
    如果操作失败，调用reject并传入reason
  */
})

```

我们先实现构造函数的框架如下：

```javascript

function Promise(executor) {
  var self = this
  self.status = 'pending' // Promise当前的状态,也是promise初始的状态
  self.data = undefined  // Promise的值
  self.onResolvedCallback = [] // Promise resolve时的回调函数集，因为在Promise结束之前有可能有多个回调添加到它上面
  self.onRejectedCallback = [] // Promise reject时的回调函数集，因为在Promise结束之前有可能有多个回调添加到它上面

  executor(resolve, reject) // 执行executor并传入相应的参数
}


```

上面的代码基本实现了Promise构造函数的主体，但目前还有两个问题：

1. 我们给executor函数传了两个参数：resolve和reject，这两个参数目前还没有定义

2. executor有可能会出错（throw），类似下面这样，而如果executor出错，Promise应该被其throw出的值reject：

```javascript

new Promise(function(resolve, reject) {
  throw 2
})

```

所以我们需要在构造函数里定义resolve和reject这两个函数：

```javascript

function Promise(executor) {
  var self = this
  self.status = 'pending' // Promise当前的状态
  self.data = undefined  // Promise的值
  self.onResolvedCallback = [] // Promise resolve时的回调函数集，因为在Promise结束之前有可能有多个回调添加到它上面
  self.onRejectedCallback = [] // Promise reject时的回调函数集，因为在Promise结束之前有可能有多个回调添加到它上面

  function resolve(value) {
    // TODO
  }

  function reject(reason) {
    // TODO
  }

  try { // 考虑到执行executor的过程中有可能出错，所以我们用try/catch块给包起来，并且在出错后以catch到的值reject掉这个Promise
    executor(resolve, reject) // 执行executor
  } catch(e) {
    reject(e)
  }
}

```

有人可能会问，resolve和reject这两个函数能不能不定义在构造函数里呢？考虑到我们在executor函数里是以resolve(value)，reject(reason)的形式调用的这两个函数，而不是以resolve.call(promise, value)，reject.call(promise, reason)这种形式调用的，所以这两个函数在调用时的内部也必然有一个隐含的this，也就是说，要么这两个函数是经过bind后传给了executor，要么它们定义在构造函数的内部，使用self来访问所属的Promise对象。所以如果我们想把这两个函数定义在构造函数的外部，确实是可以这么写的：


```javascript

function resolve() {
  // TODO
}
function reject() {
  // TODO
}
function Promise(executor) {
  try {
    executor(resolve.bind(this), reject.bind(this))
  } catch(e) {
    reject.bind(this)(e)
  }
}

```

但是众所周知，bind也会返回一个新的函数，这么一来还是相当于每个Promise对象都有一对属于自己的resolve和reject函数，就跟写在构造函数内部没什么区别了，所以我们就直接把这两个函数定义在构造函数里面了。不过话说回来，如果浏览器对bind的所优化，使用后一种形式应该可以提升一下内存使用效率。

>另外我们这里的实现并没有考虑隐藏this上的变量，这使得这个Promise的状态可以在executor函数外部被改变，在一个靠谱的实现里，构造出的Promise对象的状态和最终结果应当是无法从外部更改的。

接下来，我们实现resolve和reject这两个函数

```javascript

function Promise(executor) {
  // ...

  function resolve(value) {
    if (self.status === 'pending') {
      self.status = 'resolved'
      self.data = value
      for(var i = 0; i < self.onResolvedCallback.length; i++) {
        self.onResolvedCallback[i](value)
      }
    }
  }

  function reject(reason) {
    if (self.status === 'pending') {
      self.status = 'rejected'
      self.data = reason
      for(var i = 0; i < self.onRejectedCallback.length; i++) {
        self.onRejectedCallback[i](reason)
      }
    }
  }

  // ...
}

```

基本上就是在判断状态为pending之后把状态改为相应的值，并把对应的value和reason存在self的data属性上面，之后执行相应的回调函数，逻辑很简单，这里就不多解释了。


#### then方法

Promise对象有一个then方法，用来注册在这个Promise状态确定后的回调，很明显，then方法需要写在原型链上。then方法会返回一个Promise，关于这一点，Promise/A+标准并没有要求返回的这个Promise是一个新的对象，但在Promise/A标准中，明确规定了then要返回一个新的对象，目前的Promise实现中then几乎都是返回一个新的Promise(详情)对象，所以在我们的实现中，也让then返回一个新的Promise对象。

关于这一点，我认为标准中是有一点矛盾的：

标准中说，如果promise2 = promise1.then(onResolved, onRejected)里的onResolved/onRejected返回一个Promise，则promise2直接取这个Promise的状态和值为己用，但考虑如下代码：

```javascript

promise2 = promise1.then(function foo(value) {
  return Promise.reject(3)
})

```

此处如果foo运行了，则promise1的状态必然已经确定且为resolved，如果then返回了this（即promise2 === promise1），说明promise2和promise1是同一个对象，而此时promise1/2的状态已经确定，没有办法再取Promise.reject(3)的状态和结果为己用，因为Promise的状态确定后就不可再转换为其它状态。

另外每个Promise对象都可以在其上多次调用then方法，而每次调用then返回的Promise的状态取决于那一次调用then时传入参数的返回值，所以then不能返回this，因为then每次返回的Promise的结果都有可能不同。

下面我们来实现then方法：

```javascript

// then方法接收两个参数，onResolved，onRejected，分别为Promise成功或失败后的回调
Promise.prototype.then = function(onResolved, onRejected) {
  var self = this
  var promise2

  // 根据标准，如果then的参数不是function，则我们需要忽略它，此处以如下方式处理
  onResolved = typeof onResolved === 'function' ? onResolved : function(v) {}
  onRejected = typeof onRejected === 'function' ? onRejected : function(r) {}

  if (self.status === 'resolved') {
    return promise2 = new Promise(function(resolve, reject) {

    })
  }

  if (self.status === 'rejected') {
    return promise2 = new Promise(function(resolve, reject) {

    })
  }

  if (self.status === 'pending') {
    return promise2 = new Promise(function(resolve, reject) {

    })
  }
}

```

Promise总共有三种可能的状态，我们分三个if块来处理，在里面分别都返回一个new Promise。

根据标准，我们知道，对于如下代码，promise2的值取决于then里面函数的返回值：

```javascript

promise2 = promise1.then(function(value) {
  return 4
}, function(reason) {
  throw new Error('sth went wrong')
})

```

如果promise1被resolve了，promise2的将被4 resolve，如果promise1被reject了，promise2将被new Error('sth went wrong') reject，更多复杂的情况不再详述。

所以，我们需要在then里面执行onResolved或者onRejected，并根据返回值(标准中记为x)来确定promise2的结果，并且，如果onResolved/onRejected返回的是一个Promise，promise2将直接取这个Promise的结果：

```javascript
Promise.prototype.then = function(onResolved, onRejected) {
  var self = this
  var promise2

  // 根据标准，如果then的参数不是function，则我们需要忽略它，此处以如下方式处理
  onResolved = typeof onResolved === 'function' ? onResolved : function(value) {}
  onRejected = typeof onRejected === 'function' ? onRejected : function(reason) {}

  if (self.status === 'resolved') {
    // 如果promise1(此处即为this/self)的状态已经确定并且是resolved，我们调用onResolved
    // 因为考虑到有可能throw，所以我们将其包在try/catch块里
    return promise2 = new Promise(function(resolve, reject) {
      try {
        var x = onResolved(self.data)
        if (x instanceof Promise) { // 如果onResolved的返回值是一个Promise对象，直接取它的结果做为promise2的结果
          x.then(resolve, reject)
        }
        resolve(x) // 否则，以它的返回值做为promise2的结果
      } catch (e) {
        reject(e) // 如果出错，以捕获到的错误做为promise2的结果
      }
    })
  }

  // 此处与前一个if块的逻辑几乎相同，区别在于所调用的是onRejected函数，就不再做过多解释
  if (self.status === 'rejected') {
    return promise2 = new Promise(function(resolve, reject) {
      try {
        var x = onRejected(self.data)
        if (x instanceof Promise) {
          x.then(resolve, reject)
        }
      } catch (e) {
        reject(e)
      }
    })
  }

  if (self.status === 'pending') {
  // 如果当前的Promise还处于pending状态，我们并不能确定调用onResolved还是onRejected，
  // 只能等到Promise的状态确定后，才能确实如何处理。
  // 所以我们需要把我们的**两种情况**的处理逻辑做为callback放入promise1(此处即this/self)的回调数组里
  // 逻辑本身跟第一个if块内的几乎一致，此处不做过多解释
    return promise2 = new Promise(function(resolve, reject) {
      self.onResolvedCallback.push(function(value) {
        try {
          var x = onResolved(self.data)
          if (x instanceof Promise) {
            x.then(resolve, reject)
          }
        } catch (e) {
          reject(e)
        }
      })

      self.onRejectedCallback.push(function(reason) {
        try {
          var x = onRejected(self.data)
          if (x instanceof Promise) {
            x.then(resolve, reject)
          }
        } catch (e) {
          reject(e)
        }
      })
    })
  }
}

// 为了下文方便，我们顺便实现一个catch方法
Promise.prototype.catch = function(onRejected) {
  return this.then(null, onRejected)
}
```

## 具体到$q

$q是Angular的一种内置服务，它可以使你异步地执行函数，并且当函数执行完成时它允许你使用函数的返回值（或异常）。

defer的字面意思是延迟，$q.defer() 可以创建一个deferred实例（延迟对象实例）。

deferred 实例旨在暴露派生的Promise 实例，以及被用来作为成功完成或未成功完成的信号API，以及当前任务的状态。这听起来好复杂的样子，总结$q, defer, promise三者之间的关系如下所示。

```javascript

var deferred = $q.defer();  //通过$q服务注册一个延迟对象 deferred
var promise = deferred.promise;  //通过deferred延迟对象，可以得到一个承诺promise，而promise会返回当前任务的完成结果


```

## $q的典型应用

```javascript

var defer = $q.defer();

setTimeout(function (){
    defer.resolve(10);
},5000);
console.log("1");
defer.promise.then(function(a){
    console.log(a);
});
console.log("2");

// 打印的顺序是1,2,10
// 当我们依赖的数据会在一个异步的时间点返回时，我们需要使用resolve 跟 then 来配合


```

首先来看一下，angular的$q是如何实现promise的then方法的

```javascript
// Promise.then
function Promise() {
this.$$state = { status: 0 };
}

Promise.prototype = {
    then: function(onFulfilled, onRejected, progressBack) {
        var result = new Deferred();

        this.$$state.pending = this.$$state.pending || [];
        this.$$state.pending.push([result, onFulfilled, onRejected, progressBack]);
        if (this.$$state.status > 0) scheduleProcessQueue(this.$$state);

        return result.promise;
    },
.........
```

Promise构造函数原型上维护一个方法 then ，当一个promise对象调用该方法时会往自己的成员属性 $$state 上构建一个 pending数组，然后将生成的deferred跟成功、失败回调压入pending数组。
拿上文的案例来看， defer.promise.then之后 defer.promise.$$state.pending = [[new Deferred(), function (a){console.log(a)}]];
再来看看defer.resolve发生了什么

2.Deferred.resolve

```javascript

// Deferred.resolve
Deferred.prototype = {
    resolve: function(val) {
        if (this.promise.$$state.status) return;
        if (val === this.promise) {
            this.$$reject($qMinErr(
                'qcycle',
            "Expected promise to be resolved with value other than itself '{0}'",
            val));
        }
        else {
            this.$$resolve(val);
        }

    },

    $$resolve: function(val) {
        var then, fns;

        fns = callOnce(this, this.$$resolve, this.$$reject);
        try {
            if ((isObject(val) || isFunction(val))) then = val && val.then;
            if (isFunction(then)) {
                this.promise.$$state.status = -1;
                then.call(val, fns[0], fns[1], this.notify);
            } else {
                this.promise.$$state.value = val;
                this.promise.$$state.status = 1;
                scheduleProcessQueue(this.promise.$$state);
            }
        } catch (e) {
            fns[1](e);
            exceptionHandler(e);
        }
    },
......

```
核心方法是 Deferred.$$resolve , 上文中 defer.resolve(10) 之后会走到 scheduleProcessQueue(this.promise.$$state);

```javascript

// scheduleProcessQueue
function scheduleProcessQueue(state) {
  if (state.processScheduled || !state.pending) return;
  state.processScheduled = true;
  nextTick(function() { processQueue(state); });
}

```

nextTick方法定义在这里

```javascript

// nextTick
function qFactory(nextTick, exceptionHandler) {
  var $qMinErr = minErr('$q', TypeError);
  .....


```

再往上看谁调用了qFactory

```javascript

// qFactory
function $$QProvider() {
  this.$get = ['$browser', '$exceptionHandler', function($browser, $exceptionHandler) {
    return qFactory(function(callback) {
      $browser.defer(callback);
    }, $exceptionHandler);
  }];
}

```

没错就是我们使用的 $q 服务，也就是$q在初始化的时候会 生成 nextTick = function(callback) {$browser.defer(callback);}
$browser.defer干了什么事？

```javascript
// $browser.defer
self.defer = function(fn, delay) {
  var timeoutId;
  outstandingRequestCount++;
  timeoutId = setTimeout(function() {
    delete pendingDeferIds[timeoutId];
    completeOutstandingRequest(fn);
  }, delay || 0);
  pendingDeferIds[timeoutId] = true;
  return timeoutId;
};

```

没错，很简单，就是一个定时器。将定时器时间默认设置为0的意图很明显，使当前任务脱离主线程，在事件队列执行时再做处理，也就是所谓的defer延时
回到 scheduleProcessQueue，被手动从主线程中移除的任务是 nextTick(function() { processQueue(state); });
看看 processQueue 是干嘛的

```javascript
// processQueue
function processQueue(state) {
  var fn, promise, pending;

  pending = state.pending;
  state.processScheduled = false;
  state.pending = undefined;
  for (var i = 0, ii = pending.length; i < ii; ++i) {
    promise = pending[i][0];
    fn = pending[i][state.status];
    try {
      if (isFunction(fn)) {
        promise.resolve(fn(state.value));
      } else if (state.status === 1) {
        promise.resolve(state.value);
      } else {
        promise.reject(state.value);
      }
    } catch (e) {
      promise.reject(e);
      exceptionHandler(e);
    }
  }
}

```

没错，这个就是Promise处理最核心的部位。当主线程执行完毕开始处理事件队列时，processQueue 开始执行， 它会将 state.pending 队列(没错，队列里的内容就是我们调用then()方法时一个个压入的)遍历然后依次调用
假如有这样一段代码
promiseO.then(funcA).then(funcB);
那么执行后会发生这样一个变化

>promiseO.then(funcA) ---> promiseO.$$state.pengding = [[deferredA, funcA]]; return deferredA.promise;  
deferredA.promise.then(funcB) ---> deferredA.promise.$$state.pending = [[deferredB, funcB]]; return deferredB.promise;  

>// 当各个promise被依次resolve了
// 1. 首先是promiseO.resolve(10) ---> promiseO.$$state.value = 10;promiseO.$$state.status = 1;scheduleProcessQueue(promiseO.$$state);
// 2. scheduleProcessQueue最后走到processQueue方法，会执行promiseO.$$state.pengding队列，这时候会执行 deferredA.resolve(funcA(promiseO.$$state.value))
// 3. 假设funcA(promiseO.$$state.value)返回20，那么就是 deferredA.resolve(20)。这时候就又回到了步骤1
// 依次执行后then链上所有方法均会执行

then的核心思想是它会往当前promise的pending队列中压入then链下一个promise对象的Deferred(promise=Deferred.promise),然后通过这种节点关系构成整个then链

介绍完promise.then我们再来介绍一下$q.all([promises]).then(funcT)，来看看他是怎么实现当所有promise被resolve之后再走向下一步的吧

## $q.all

```javascript
// $q.all
function all(promises) {
  var deferred = new Deferred(),
      counter = 0,
      results = isArray(promises) ? [] : {};

  forEach(promises, function(promise, key) {
    counter++;
    when(promise).then(function(value) {
      if (results.hasOwnProperty(key)) return;
      results[key] = value;
      if (!(--counter)) deferred.resolve(results);
    }, function(reason) {
      if (results.hasOwnProperty(key)) return;
      deferred.reject(reason);
    });
  });

  if (counter === 0) {
    deferred.resolve(results);
  }

  return deferred.promise;
}

```
很简单，forEach promises列表之后，在每个promise的then链加一个回调，关键代码在这一行
if (!(--counter)) deferred.resolve(results);
每一个promise成功执行了then回调之后会--counter,如果某个promise是最后一个被resolve的则将所有的results resolve，这个时候他的then链就开始执行了

## $q.when

```javascript
//$q.when
var when = function(value, callback, errback, progressBack) {
  var result = new Deferred();
  result.resolve(value);
  return result.promise.then(callback, errback, progressBack);
};

```

$q.when(val/promiseLikeObj)的用处很简单，就是将一个简单对象/或promiseLike的对象包装成$q信任的promise对象并将值加入到then链中。





