---
title: import还是require？JS依赖处理的前世今生
tags: [代码]
---

## 引言

在写代码的过程中，我们不可能只用一个文件就装下所有的代码，必然要多文件协同配合，此时就涉及到了依赖。不同的程序语言都有不同的依赖处理方式，例如C语言中的`#include`和Cmake，或者Python中的`import`。哪种语言的处理方式最好可能不太好说，但是要说哪种语言最差，要我说就当属Javascript。依赖处理这么基本的问题，JavaScript折腾了几十年，依旧会有各种各样的问题。那么到底为什么会出现这种问题呢，我们首先要梳理一下历史的脉络。

## Part1 前Node时代

众所周知，JavaScript在设计之初就是为浏览器服务的。所以Javascript的代码天然是需要被HTML来引用的。如果你看过很多基础的前端三件套课程，或者那种很古老的网页设计兴趣班课程（我是在初中时的网页设计活动课中学到的），那么你对下面的代码一定不陌生：

```html
<html>
    <head>
        <title>网页标题</title>
        <script src='script.js'></script>
    </head>
    <body>
    </body>
</html>
```

如果有多个代码，那么它就会按照先后顺序执行。多文件之间的依赖？不存在的。你在函数外使用`var`声明的变量是自动被挂载到`window`对象上的，后续的代码想用，直接就能拿过来[^1]。

```js
// global.js
var global1 = "I'm a global!";
var global2 = "So am I!";

// other js-file
function testGlobal () {
    alert(global1);
}
```

这就类似于早期C语言的`#include`，如果你只是简单的写一点脚本的话倒还好。后来随着JS功能的增强，代码也变得越来越复杂，同时很多第三方库的涌现让代码量激增，如果还用这种方式来写的话，不光变量污染会非常严重，同时项目也非常难以维护。那你一定会问，那浏览器或 EcmaScript 组织为什么不出手？其实 1999–2008 年间规范几乎停滞，组织是完全的停摆状态，所以我们只能看民间的自救了，也就是Node的诞生。

## Part2 Node与CommonJS

2009年前后，V8引擎的出现和Commonjs的提出，第一次让JavaScript这门语言脱离浏览器环境来运行，在众多实现中，最出名的还是Ryan Dahl创造的Node。既然JavaScript这门语言具备了独立运行的能力，那么我们就顺理成章的把依赖处理这件事提到了日程。在CommonJS标准中，采用了`require`这一关键字来导入一个包，下面这段代码就是一个简单的node的依赖引用的例子：

```js
// math.js
exports.add = (a, b) => a + b;

// main.js
const { add } = require('./math');
console.log(add(2, 3));
```

我们让`exports`对象挂载我们需要用的变量或函数，使用的时候直接`require`进来你需要的成员就好，不会污染你的变量。如果你嫌这种方式要导入所有东西很麻烦的话，还可以直接给`module.exports`对象赋值，附成你所需要的变量集合的Object，这样调用方可以整体接收，也可以自行解构实现“按需”。例如下面这个例子：

```js
// utils.js
function add(a, b) { return a + b; }
function mul(a, b) { return a * b; }
function div(a, b) { return a / b; }

module.exports = { add, mul, div };

// main.js
const utils = require('./utils'); 
console.log(utils.add(2, 3));
```

由于在node刚出现的时候，js还没有`class`语法糖，社区普遍用函数/原型模拟类，因此很多官方库的主体都是一个巨大的Object，同时由于核心模块需要一次抛出几十种 API，采用整体对象导出最方便，所以统一使用 module.exports = obj，也因此官方包都是走`module.exports`的路线，例如`const fs = require('fs')`。

这一时期的node可以说在后端大放异彩，但是我们忽视了一个重要的问题：Commonjs并不是浏览器标准，只是node自己遵循的标准，浏览器是读不了`require`这种代码的。所以浏览器环境下的jS依旧停滞不前。事情要等到ES6出现之后才有转机。

## Part3 ES6与前端工程化

2015年推出的ES6，才终于在浏览器层面增加了`import`和`export`关键字作为依赖处理和模块化来使用。要写一个ES6的模块，首先你要在HTML中显示声明这个js文件是一个模块，也就是在`<script>`标签中添加`type="module"`属性。

## 参考链接
+ [^1]: 一篇StarkOverFlow的文章回答给出了这段示例代码 https://stackoverflow.com/questions/944273/how-to-declare-a-global-variable-in-a-js-file