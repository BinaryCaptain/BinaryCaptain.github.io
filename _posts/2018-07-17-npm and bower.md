---
layout: post
keywords: npm bower
description: npm和bower的区别比较
title: "npm bower"
tags: [npm]
date: 2018-07-17
categories: npm
cover: 'https://binarycaptain.github.io/assets/img/npm.jpg'
tags: npm
subtitle: 'npm'

---


## npm和bower的区别比较

众所周知，npm（Node Package Manager）是nodejs时代不可或缺的最好的包管理器，现在已经随nodejs官方包同时会安装到你的设备上去。只要给项目书写好package.json放于项目根目录，在重新部署之时只需要执行 npm install 一行简单的命令，所有相关的依赖就能够自动安装到项目目录下面，并且还能很方便的对不同项目的不同依赖包版本进行良好、统一的管理。

重点来说说NPM和Twitter推出的名为 Bower 的包管理器之间到底有什么样的关系和区别呢？（Bower的官网写到，Bower 是 “A package manager for the web” ，难道说NPM就不是了嘛）。

其实，在实际项目中，NPM和Bower都会被运用进去。并且Bower的安装和升级全都依赖于NPM，使用如下命令就可以全局安装Bower
npm install -g bower 之后你就可以使用 bower install 类似于NPM的方式，对于当前项目进行前端依赖的相关管理。使用起来和NPM一样方便快捷。

其中，与NPM最大的区别在于，NPM主要运用于Node.js项目的内部依赖包管理，安装的模块位于项目根目录下的node_modules文件夹内。而Bower大部分情况下用于前端开发，对于CSS/JS/模板等内容进行依赖管理，依赖的下载目录结构可以自定义。

有人可能会问，为何不用NPM一个工具对前后端进行统一的依赖管理呢？ 实际上，因为npm设计之初就采用了的是嵌套的依赖关系树，这种方式显然对前端不友好；而Bower则采用扁平的依赖关系管理方式，使用上更符合前端开发的使用习惯。

不过，现在越来越多出名的js依赖包可以跨前后端共同使用，所以Bower和NPM上面有不少可以通用的内容。实际项目中，我们可以采用NPM作用于后端；Bower作用于前端的组合使用模式。让前后端公用开发语言的同时，不同端的开发工程师能够更好地利用手上的工具提升开发效率。

不过随着Common.js的发展，相信以后npm会逐渐取代掉bower。