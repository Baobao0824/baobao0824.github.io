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


## 参考链接
+ [^1]: 一篇StarkOverFlow的文章回答给出了这段示例代码 https://stackoverflow.com/questions/944273/how-to-declare-a-global-variable-in-a-js-file