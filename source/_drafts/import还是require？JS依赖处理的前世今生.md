---
title: import还是require？JS模块化的前世今生
tags: [代码]
categories: 代码
---

## 引言

在写代码的过程中，我们不可能只用一个文件就装下所有的代码，必然要多文件协同配合，此时就涉及到了依赖和模块化。不同的程序语言都有不同的依赖处理方式，例如C语言中的`#include`和Cmake，或者Python中的`import`。哪种语言的处理方式最好可能不太好说，但是要说哪种语言最差，要我说就当属Javascript。依赖处理这么基本的问题，JavaScript折腾了几十年，依旧会有各种各样的问题。那么到底为什么会出现这种问题呢，我们首先要梳理一下历史的脉络。

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

## Part3 ES6模块及其带来的问题

2015年推出的ES6，才终于在浏览器层面增加了`import`和`export`关键字作为依赖处理和模块化来使用。要写一个ES6的模块，首先你要在HTML中显示声明这个js文件是一个模块，也就是在`<script>`标签中添加`type="module"`属性。由于有了`type=module`来区分普通的js和模块，因此变量再也不会全部泄露到`window`上了，在浏览器层面实现了按需导出。

```js
// 你可以直接在变量前面加 export
export const add = (a, b) => a + b;

// 你也可以像这样在最后统一导出你想要的变量
const add = (a, b) => a + b;
export {add}
```

```html
<script type="module">
  import { add } from './math.js';
  console.log(add(2, 3));   // 5
</script>
```

这种我们叫做具名导出，和Commonjs标准不同的是，ES6还提供了“默认导出”这种崭新的方式，这样的话你在使用的时候就不用记到底是哪个变量了，自己起就可以，不过要注意的是一个模块只能有一种默认导出。

```js
// math.js
export default function add(a, b) {       // 默认导出
  return a + b;
}
// main.js
import addc from './math.js';      // 默认导入随意起名
console.log(addc(1, 2)); // 3
```

如果你以为这样就万事大吉了，那可就错了。现在又有了一个新问题：在2009年到2015年这六年间，超过30多万的包都以commonJs的格式进行分发，老百姓只知道`require`，不知道`import`，而浏览器呢，恰好反过来，很显然老百姓用的node是不能等着让高贵的es老爷来适配的，只能社区自己想办法让node向浏览器看齐了，想办法将各种标准统一起来，统一打包成浏览器能够识别的数据类型，这就是Webpack被发明的原因。


## Part4 Webpack的前世今生

Webpack并不是为了解决上文中问题而生的，但是它确实是最广受支持的解决方案。

最初的webpack只是想做一个把node使用的Commonjs库引入浏览器中的构建工具，这样浏览器就可以使用很多npm库了，同时也能反过来促进前端库的开发。最早的Webpack只做一件事：在编译阶段把所有源码包进一个自执行函数，内部实现一套微型 CommonJS 运行时，最后生成一个（或多个）纯 ES5 的 bundle.js，用传统 <script src="bundle.js"></script> 引入即可跑在浏览器上。


## 参考链接
+ [^1]: 一篇StarkOverFlow的文章回答给出了这段示例代码 https://stackoverflow.com/questions/944273/how-to-declare-a-global-variable-in-a-js-file