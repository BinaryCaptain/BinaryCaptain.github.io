---
layout: post
keywords: js
description: array
title: "数组扁平化"
tags: [js array]
date: 2019-03-16
categories: 数组 扁平化
cover: 'https://binarycaptain.github.io/assets/img/js.jpg'
tags: 数组扁平化
subtitle: '数组扁平化'
---

## 扁平化

数组的扁平化，就是将一个嵌套多层的数组 array (嵌套可以是任何层数)转换为只有一层的数组。

举个例子，假设有个名为 flatten 的函数可以做到数组扁平化，效果就会如下：

```javascript

var  arr = [1, [2, [3, 4]]];
console.log(flatten(arr));

```

知道了效果是什么样的了，我们可以去尝试着写这个 flatten 函数了

## 递归

我们最一开始能想到的莫过于循环数组元素，如果还是一个数组，就递归调用该方法：

```javascript

var arr = [1, [2, [3, 4]]];

function flatten(arr) {
    var result = [];
    for (var i = 0, len = arr.length; i < len; i++) {
        if (Array.isArray(arr[i])) {
            result = result.concat(flatten(arr[i]))
        }
        else {
            result.push(arr[i])
        }
    }
    return result;
}


console.log(flatten(arr))

```

## toString 

如果数组的元素都是数字，那么我们可以考虑使用 toString 方法，因为：

```javascript

[1, [2, [3, 4]]].toString() // "1,2,3,4"

```

调用 toString 方法，返回了一个逗号分隔的扁平的字符串，这时候我们再 split，然后转成数字不就可以实现扁平化了吗？


## reduce 

既然是对数组进行处理，最终返回一个值，我们就可以考虑使用 reduce 来简化代码：

```javascript

function flattern(arr) {
    arr.reduce(function(prev,next) {
        return prev.concat(Array.isArray(next) ? flattern(next) : next)
    },[])
}

console.log(flattern(arr))

```


ES6 增加了扩展运算符，用于取出参数对象的所有可遍历属性，拷贝到当前对象之中：

```javascript

var arr = [1, [2, [3, 4]]];
console.log([].concat(...arr)); // [1, 2, [3, 4]]

```

我们用这种方法只可以扁平一层，但是顺着这个方法一直思考，我们可以写出这样的方法：

```javascript

var arr = [1, [2, [3, 4]]];

function flattern(arr) {

    while(arr.some((item) => Array.isArray(item))) {
        arr = [].concat(...arr)
    }

    return arr;
}

console.log(flatten(arr))

```

那么如何写一个抽象的扁平函数，来方便我们的开发呢，所有又到了我们抄袭 underscore 的时候了~

```javascript

function flatten(input, shallow, strict, output) {

    // 递归使用的时候会用到output
    output = output || [];
    var idx = output.length;

    for (var i = 0, len = input.length; i < len; i++) {

        var value = input[i];
        // 如果是数组，就进行处理
        if (Array.isArray(value)) {
            // 如果是只扁平一层，遍历该数组，依此填入 output
            if (shallow) {
                var j = 0, len = value.length;
                while (j < len) output[idx++] = value[j++];
            }
            // 如果是全部扁平就递归，传入已经处理的 output，递归中接着处理 output
            else {
                flatten(value, shallow, strict, output);
                idx = output.length;
            }
        }
        // 不是数组，根据 strict 的值判断是跳过不处理还是放入 output
        else if (!strict){
            output[idx++] = value;
        }
    }

    return output;

}

```


































