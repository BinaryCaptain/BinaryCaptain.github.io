---
layout: post
keywords: blog
description: blog
title: "react与新浪微博尤物志技术实践"
tags: [react]
date: 2018-02-03
categories: react
cover: 'https://binarycaptain.github.io/assets/img/react.jpg'
tags: react
subtitle: 'react技术实践'

---


## react介绍

React 项目是Facebook的内部项目，因为该公司对市场上所有JavaScript框架，都不满意，就决定自己写一套，用来架设
Instagram 的网站。做出来以后，发现这套东西很好用，就在2013年5月开源了。

由于 React的设计思想极其独特，属于革命性创新，性能出众，代码逻辑却非常简单。所以，越来越多的人开始关注和使用，认为它可能是将来 Web 开发的主流工具。

这个项目本身也越滚越大，从最早的UI引擎变成了一整套前后端通吃的 Web App 解决方案。衍生的 React Native 项目，目标更是宏伟，希望用写WebApp的方式去写NativeApp。如果能够实现，整个互联网行业都会被颠覆，因为同一组人只需要写一次 UI ，就能同时运行在服务器、浏览器和手机。React主要用于构建UI。

你可以在React里传递多种类型的参数，如声明代码，帮助你渲染出UI、也可以是静态的HTML DOM元素、也可以传递动态变量、甚至是可交互的应用组件。

特点：

1. 声明式设计：React采用声明范式，可以轻松描述应用。
2. 高效：React通过对DOM的模拟，最大限度地减少与DOM的交互。
3. 灵活：React可以与已知的库或框架很好地配合。

## react 尤物志技术实践

尤物志是微博旗下的电商版块，第一版的尤物志开发工作我并没有亲自参与，主要参与第二次的改版工作以及重构工作。下面介绍一下尤物志的技术栈。用到的主要技术有NodeJS+React+Webpack+ES6+Babel+SASS，其中核心是react技术，首先从react组件说起，任何一个复杂的应用，都是由一个简单的应用发展而来的，当应用还很简单的时候，因为功能少，可能只需要一个组件就够了，但是，随着功能的增加，把越来越多的功能放在一个组件中就会显得臃肿和难以管理。就和一个人专注做一件事情一样，也应该尽量保持一组件只做一件事情，当开发者发现一个组件功能太多，代码量太大的时候，就要考虑拆分这个组件，
用多个小的组件来代替，每个小的组件只关注实现单个功能，但是这些功能组合起来，也能够满足复杂的实际业务需求。这就是“分而治之”的策略，把问题你分解为多个小的问题，这样既绕你故意解决也方便维护。虽然这是一个好策略，但是也不能滥用，只有必要的时候采取拆分组件，不然可能得不偿失。

根据软件设计的通则，组件的划分为应当为高内聚，低耦合的原则。

高内聚指的是把逻辑紧密相关的内容放在一个组件当中，用户界面无外乎内容，交互行为和样式。传统上，内容上由HTML表示，交互行为放在JavaScript代码文件中，样式放在css文件当中进行定义。虽然是一个功能，但是却要放在三个文件当中，不符合高内聚的原则。react却不是这样，在react当中，这些东西都可以放在一个文件当中，因此，react天生具有高内聚的特点。

具体到尤物志项目当中，是根据不同的功能来划分组件的，对于仅仅是样式或者内容上有区别的功能模块，尽量复用前面已经开发好的组件，而不是再去重新去开发，做到一个模块不管放到什么地方都可以正常使用，对于后期维护都具有重要的意义。

尤物志项目中涉及到最多的就是父子组件之间的通信，而父子组件的之间的通信主要是父组件向子组件传递信息，父组件向子组件传递信息，主要使用props进行传递，子组件通过this.props.属性名来获取之前在父组件中已经定义好的信息，如果是层次较多，比如父元素向子元素的子元素(孙子元素)传递信息，这个时候比较笨的方法就是利用props一层一层往下传递，但是这样的如果嵌套不是太深的话还可以使用，如果嵌套层级太深的话，就不再适用，这个时候我们就要使用context来进行数据的传递，下面看一个案列：

```javascript
<span style="font-size:18px;">var Button = React.createClass({
  // 必须指定context的数据类型
  contextTypes: {
    color: React.PropTypes.string
  },
  render: function() {
    return (
      <button style={{background: this.context.color}}>
        {this.props.children}
      </button>
    );
  }
});

var Message = React.createClass({
  render: function() {
    return (
      <div>
        {this.props.text} <Button>Delete</Button>
      </div>
    );
  }
});

var MessageList = React.createClass({
  //父组件要定义 childContextTypes 和 getChildContext()
  childContextTypes: {
    color: React.PropTypes.string
  },
  getChildContext: function() {
    return {color: "purple"};
  },
  render: function() {
    var children = this.props.messages.map(function(message) {
      return <Message text={message.text} />;
    });
    return <div>{children}</div>;
  }
});</span>
```

以上代码中通过添加 childContextTypes 和 getChildContext() 到 第一层组件MessageList （ context 的提供者），React 自动向下传递数据然后在组件中的任意组件（也就是说任意子组件，在此示例代码中也就是 Button ）都能通过定义 contextTypes（必须指定context的数据类型） 访问 context 中的数据。这样就不需要通过第二层组件进行传递了。

指定数据并要将数据传递下去的父组件要定义 childContextTypes 和 getChildContext() ；想要接收到数据的子组件 必须定义 contextTypes 来使用传递过来的 context 。

使用这种方法减少了之前那种多余的嵌套，大大优化了代码，有利于后期的维护。

虽然项目中并没有涉及到子组件向父组件传递信息，但是这里还是要简单的讲一下,案列如下：

```javascript

<span style="font-size:18px;">// 父组件
var MyContainer = React.createClass({
  getInitialState: function () {
    return {
      checked: false
    };
  },
  onChildChanged: function (newState) {
    this.setState({
      checked: newState
    });
  },
  render: function() {
    var isChecked = this.state.checked ? 'yes' : 'no';
    return (
      <div>
        <div>Are you checked: {isChecked}</div>
        <ToggleButton text="Toggle me"
          initialChecked={this.state.checked}
          callbackParent={this.onChildChanged}
          />
      </div>
    );
  }
});

// 子组件
var ToggleButton = React.createClass({
  getInitialState: function () {
    return {
      checked: this.props.initialChecked
    };
  },
  onTextChange: function () {
    var newState = !this.state.checked;
    this.setState({
      checked: newState
    });

    //这里将子组件的信息传递给了父组件
    this.props.callbackParent(newState);
  },
  render: function () {
    // 从（父组件）获取的值
    var text = this.props.text;
    // 组件自身的状态数据
    var checked = this.state.checked;
        //onchange 事件用于单选框与复选框改变后触发的事件。
    return (
        <label>{text}: <input type="checkbox" checked={checked}                 onChange={this.onTextChange} /></label>
    );
  }
});</span>

```
以上例子中，在父组件绑定callbackParent={this.onChildChanged}，在子组件利用this.props.callbackParent(newState),触发了父级的的this.onChildChanged方法，进而将子组件的数据（newState）传递到了父组件。
这样做其实是**依赖 props 来传递事件的引用，并通过回调的方式来实现的**

那么还有一种传递是没有任何嵌套关系的组件之间传值（比如：兄弟组件之间传值）

这个例子需要引入一个PubSubJS 库，通过这个库你可以订阅的信息，发布消息以及消息退订。
 具体可参考下面的内容
[PubSubJS](http://blog.csdn.net/u011439689/article/details/51955991)

```javascript
<span style="font-size:18px;">// 定义一个容器（将ProductSelection和Product组件放在一个容器中）
var ProductList = React.createClass({
    render: function () {
      return (
        <div>
          <ProductSelection />
          <Product name="product 1" />
          <Product name="product 2" />
          <Product name="product 3" />
        </div>
      );
    }
});
// 用于展示点击的产品信息容器
var ProductSelection = React.createClass({
  getInitialState: function() {
    return {
      selection: 'none'
    };
  },
  componentDidMount: function () {
    //通过PubSub库订阅一个信息
    this.pubsub_token = PubSub.subscribe('products', function (topic, product) {
      this.setState({
        selection: product
      });
    }.bind(this));
  },
  componentWillUnmount: function () {
    //当组件将要卸载的时候，退订信息
    PubSub.unsubscribe(this.pubsub_token);
  },
  render: function () {
    return (
      <p>You have selected the product : {this.state.selection}</p>
    );
  }
});

var Product = React.createClass({
  onclick: function () {
    //通过PubSub库发布信息
    PubSub.publish('products', this.props.name);
  },
  render: function() {
    return <div onClick={this.onclick}>{this.props.name}</div>;
  }
});</span>

```

ProductSelection和Product本身是没有嵌套关系的，而是兄弟层级的关系。但通过在ProductSelection组件中订阅一个消息，在Product组件中又发布了这个消息，使得两个组件又产生了联系，进行传递的信息。所以根据我个人的理解，当两个组件没有嵌套关系的时候，也要通过全局的一些事件等，让他们联系到一起，进而达到传递信息的目的。



## react 声明周期介绍

实例化
首次实例化

getDefaultProps
getInitialState
componentWillMount
render
componentDidMount

实例化完成后的更新

getInitialState
componentWillMount
render
componentDidMount

存在期
组件已存在时的状态改变

componentWillReceiveProps
shouldComponentUpdate
componentWillUpdate
render
componentDidUpdate
销毁&清理期
componentWillUnmount
说明
生命周期共提供了10个不同的API。

现在我们通常用ES6的方法创建组件，这个时候并不会发生1和2这2个过程。

1.getDefaultProps
作用于组件类，只调用一次，返回对象用于设置默认的props，对于引用值，会在实例中共享。

2.getInitialState
作用于组件的实例，在实例创建时调用一次，用于初始化每个实例的state，此时可以访问this.props。

3.componentWillMount
在完成首次渲染之前调用，此时仍可以修改组件的state。它既可以在浏览器端被调用，也可以在服务器端被调用。

4.render
必选的方法，创建虚拟DOM，该方法具有特殊的规则：

只能通过this.props和this.state访问数据
可以返回null、false或任何React组件
只能出现一个顶级组件（不能返回数组）
不能改变组件的状态
不能修改DOM的输出

5.componentDidMount
真实的DOM被渲染出来后调用，在该方法中可通过this.getDOMNode()访问到真实的DOM元素。此时已可以使用其他类库来操作这个DOM。

***在服务端中，该方法不会被调用***这是它和componentWillMount的区别。

6.componentWillReceiveProps
很多说法是组件接收到新的props时调用，其实这是不正确的，实际上是只要父组件的render函数被调用，在render函数里面被渲染的子组件就会经历更新过程，不管父组件传给子组件的props是否发生改变，都会触发此函数，下面是接收到新的props的时候，并将其作为参数nextProps使用，此时可以更改组件props及state。

```javascript
  componentWillReceiveProps: function(nextProps) {
        if (nextProps.bool) {
            this.setState({
                bool: true
            });
        }
    }
```
7.shouldComponentUpdate
组件是否应当渲染新的props或state，返回false表示跳过后续的生命周期方法，通常不需要使用以避免出现bug。在出现应用的瓶颈时，可通过该方法进行适当的优化。

在首次渲染期间或者调用了forceUpdate方法后，该方法不会被调用

8.componentWillUpdate
接收到新的props或者state后，进行渲染之前调用，此时不允许更新props或state。

9.componentDidUpdate
完成渲染新的props或者state后调用，此时可以访问到新的DOM元素。

10.componentWillUnmount
组件被移除之前被调用，可以用于做一些清理工作，在componentDidMount方法中添加的所有任务都需要在该方法中撤销，比如创建的定时器或添加的事件监听器。



























