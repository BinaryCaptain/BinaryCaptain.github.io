---
layout: post
keywords: http
description: http
title: "http request"
tags: [http]
date: 2017-03-01
categories: http
cover: 'https://binarycaptain.github.io/assets/img/http.png'
tags: http
subtitle: 'http请求与常见状态码'

---

## HTTP请求
HTTP请求的是资源(Resource)

资源由URL决定

资源分为静态资源和动态资源

## 常见的状态码规则

![](https://binarycaptain.github.io/assets/img/http.png)

面试中比较常问的就是状态码200和304的区别，304就代表浏览器直接从本地缓存当中去读取需要的资源，304与强制缓存所对应，200则需要重新向服务器发起请求，重新拉取资源，200与协商缓存对应。

还有比较常见的401状态码，401状态码就是你在没有登陆的情况下想要访问需要登陆才能获取到的资源的时候，这个时候服务器就会返回401状态码，还有一个比较常见的就是403状态码(Forbidden),比如我们去访问一个网站，然后不停刷新的话，经过后台判断为恶意请求，这个时候就会返回403状态码，限制你的访问频率u，

## 浏览器与状态码

F5和Ctrl+R一样都是刷新，就是刷新页面。强制刷新Ctrl+F5(强刷)就是不允许使用缓存，所有资源都重新载入，比如你看知乎发现页面上的图片只加载了一半就停掉了，如果只刷新可能还是用的缓存里的这半张照片，那么就可以按个Ctrl+F5。

## post请求的Content-type类型

![](https://binarycaptain.github.io/assets/img/http-1.png)