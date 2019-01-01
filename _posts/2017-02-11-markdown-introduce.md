---
layout: post
keywords: blog
description: blog
title: "markdown 语法"
tags: [markdown]
date: 2017-02-11
categories: markdown
cover: 'https://binarycaptain.github.io/assets/img/markdown.jpg'
tags: markdown
subtitle: 'markdown的使用'

---

>引言：Markdown是一种轻量级的标记语言，轻到你甚至可以不叫他语言，因为Markdown很容易上手，就是简单地记住几个常用的标签用法就OK了，Markdown有诸多好处：专注于文字，简单，高效.

## 以下的符号请全部以英文输入法为准！！！

下面来详细说说那几个标签（真的只有几个）：

代码（开发人员专用）:

前提是输入法在英文模式下。代码写在 `这里是你要写的代码` 之间，``号（windows下）就是ESC下面的那个按键


如果是代码块，请放在六个这个符号中间，然后按Tab缩进，下面这样：

![](https://binarycaptain.github.io/assets/img/code-format.png)

常用标题

```html
# 代表h1（一级标题）
## 代表h2（二级标题）
###代表h3（三级标题）
```
效果是这样的：

#代表h1（一级标题）
##代表h2（二级标题）
###代表H3（三级标题）
链接（有两种写法）

第一种[text](url)：
```html
    [text](url)
    比如说：[laravist](https://laravist.com)
```
第二种[text][id] [id]:url：
```html

    [text][id]

    [id]:url
```
比如说：
```html
    [laravist][1]

    [1]: https://laravist.com
```

text代表你要链接的文字，URL就是连接地址，第二种的id你可以随便定，但建议是用数字就OK，而且在段落中也推荐使用第二种方法并且把
[id]:url放在段落之后；因为链接很多时候都很长，会影响编写文档的美感。而且这样反而会更高效.

插入图片![](image-url)：

跟链接的第一种方式很像

```html
   ![](image-url)
```
比如说：

```html
  ![](https://laravist.com/images/yanchao.png)
```

加粗****：

>**这是要加粗的文字**   四个星号中间就是你要加粗的内容

我觉得这一个就够了，效果：

这是要加粗的文字 四个星号中间就是你要加粗的内容

斜体** ：

```html *这是你要斜体的文字*   两个星号中间就是要斜体的文字```
也是这个就够了，效果如下：

*这是你要斜体的文字* 两个星号中间就是要斜体的文字

无序列表:*

```html
 * 无序列表内容1
    * 无序列表内容2
    * 无序列表内容3
    一个星号加一个空格，注意星号与文字之间有一个空格
```
有序列表：

```html
    1. 有序列表内容1
    2. 有序列表内容2
    3. 有序列表内容3
    一个数字加一个点，文子与点之间有一个空格
```

引用>：

```html
 >这是你要引用的内容
 ```
 向右的尖括号表示引用

 水平分割线***：

```html ***这是一条水平分割线```
用连续的三个星号表示

**总结一下：**

五个常用的而已，他们的组合也不多，就上面的介绍几个，把他们用熟练，对于个人的写作效率会有很大的提升。
>其实Markdown语法无非就是几个常用的符号： #,*,(),[], >