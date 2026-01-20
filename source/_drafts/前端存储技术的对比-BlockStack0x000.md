---
title: 前端存储技术的对比-BlockStack0x000
tags: [代码,块叠BlockStack, 前端]
categories: [块叠BlockStack]
---

> 原链接：https://www.gfbzsblog.site/main/Articles/158

## 写在前面

这是一篇个人博客中的技术文章，他主要讲解了在前端技术中如何进行数据的存储，主要是Cookie、LocalStorage和sessionStorage，以及他们三个的特性和使用场景。这是一篇比较基础的、比较适合啃的文章，第一期块叠就找他了。

## 一、存储机制的本质差异

首先是Cookie，它出现的最早，本质上是一个键值对。由于HTTP协议是无状态的，收到HTTP请求的服务器并不知道收到的请求来自什么用户，所以就需要客户端有个标记，因此Cookie就应运而生了。一般来说第一次访问某个网站的话，http请求头中是不带Cookie字段的，但是假如服务器需要cookie的话，那它就会在返回的时候带上`Set-Cookie`字段。

```text
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Set-Cookie: sessionid=abc123; Path=/; HttpOnly; SameSite=Lax
Set-Cookie: theme=dark; Max-Age=2592000; Path=/; Secure
```

一个Set-Cookie对应一个键值对，后续跟的是一些属性信息。同时浏览器接收到这个请求之后，就会将cookie放在自己的Cookie数据库下（各浏览器的实现不同），后续同站点下，客户端再发其二http请求的话，就会在请求头中携带相应的Cookie。

```text
GET /profile HTTP/1.1
Host: example.com
Cookie: sessionid=abc123; theme=dark
```
接下来是localStorage，它是HTML5标准中引入的Web Storage API的一部分，但是和用于服务端通信不同，它只存储于客户端，并且会一直存放在浏览器中，除非用户主动清除。

最后是sessionStorage，它也是Web Storage API的一部分，但是它并不进行持久化存储，只在当前标签页有效，一旦标签页被关闭，数据就会被清除，也使得它更适合用于存储一些非常临时的状态。

## 二、详细特征对比

### 可用容量大小

+ Cookie：一条Cookie（带属性的一个键值对）容量为4kb，每个域名有20条的容量限制。
+ localStorage：5-10MB（具体大小由浏览器决定），对于客户端就足够用了。
+ SessionStorage：同上，也是50-10MB

### 数据生命周期

Cookie：

+ 可以设置过期时间（`Expires`或`Max-Age`，现在后者优先级更高）。
+ 如果不设置过期时间，则为会话Cookie，关闭浏览器后失效。
+ 设置了过期时间的Cookie会一直存在，直到过期或被手动删除。

localStorage：

+ 数据永久存储，除非用户手动清除或通过代码删除。
+ 不会因为关闭浏览器而丢失。
+ 不会因为重启电脑而丢失。

sessionStorage：

+ 数据只在当前标签页有效。
+ 关闭标签页后数据立即清除。
+ 刷新页面不会清除数据。
+ 如果在同一个标签页内跳转或刷新，数据会保留。
+ 复制新的标签页，不会将sessionStorage带过去。

## （扩展）cookie的属性介绍

一条 Cookie 由“名/值”和 7 个可选属性组成，浏览器在设置（Set-Cookie 响应头 或 `document.cookie=`）时逐条解析，而读取的时候只能读取到键和值，属性是不能获取的。下面介绍一下一个Cookie拥有的所有属性

Name&Vale：

也就是键值对，这是一条Cookie中最核心的内容。例如`favorite_food=tripe`，遵循url的编码方案，所以如果有空格和非ASCII字符等，别忘了在赋值之前进行转换。

Expires：

即绝对过剩时刻，需要传一个GMT风格的字符串，如果不传的话，就是会话级别的Cookie，浏览器关闭就会失效。例如`expires=Tue, 19 Jan 2027 12:00:00 GMT`。

Max-Age：

最大时长，相对于设置时间的秒数，虽然优先级高于Expires，但是旧版IE并不支持。例如`max-age=3600`。

Domain：

+ 控制什么等级的域名可以接收该Cookie，如果不写的话，则默认“当前域名”。
+ 如果人工指定了一个域名的话的话，那么指定域名和子级域名都能获取到。
+ 注意在显示指定的时候，一定要指定成当前的域名或者其父域名，否则浏览器直接无视。
+ 例如`domain=.far.boo.com`，此时far.boo.com、a.far.boo.com都能读到，而boo.com读不到。

Path：

+ 这个属性规定了Cookie可以在哪些URL路径下随请求而携带。
+ 如果留空的话，默认是“当前路径”及其子路径。
+ 通常设为`/`，这样在整个站点中都可用了。 
+ 例如，如果设置为`path=/shop`的话，那么只有/shop，/shop/a...的请求才会携带对应的Cookie。

Secure：

布尔属性，出现则为True，如果为True则指示浏览器，只在HTTPS连接中才发送该Cookie，防止明文传输被盗。使用的时候直接添加`secure`即可。

HttpOnly  
同上，也是布尔属性，若为True则指示浏览器，禁止JS通过`document.cookie`变量对其进行读写，降低泄露风险，使用的时候直接添加`httponly`即可。

1. SameSite（RFC 6265bis，现代浏览器已支持）  
   防 CSRF 与跨站定时攻击。可取：  
   - `Strict`：完全禁止跨站发送（包括点击外链）；  
   - `Lax`：默认值，允许安全 HTTP 方法（GET、HEAD）的顶层导航带 Cookie；  
   - `None`：允许跨站，但必须同时加 `Secure`（即仅 HTTPS）。  
   例：`samesite=lax`

2. Priority（Chrome 私有扩展）  
   低、中、高。当浏览器 Cookie 数量达到上限时，先踢掉低优先级。  
   例：`priority=high`

快速记忆口诀：  
“值随名，过期分两种（Expires/Max-Age），域路径管范围，Secure 只 HTTPS，HttpOnly 防脚本，SameSite 挡 CSRF，Priority 给 Chrome 做清理。”

### 数据访问方式

在js中，cookie被绑定在了`document.cookie`变量中，通过调用或修改这个变量的值，就能够设置或读取Cookie了。我们可以通过给cookie赋值来添加一条cookie。这是MDN上的例子：

```js
document.cookie = "name=oeschger";
document.cookie = "favorite_food=tripe";
alert(document.cookie);
// 显示：name=oeschger;favorite_food=tripe
document.cookie = "favorite_food=oreo";
// 显示：name=oeschger;favorite_food=oreo
```
是的，我们每次只能获取到所有的cookie，如果需要从中筛选，就需要自己手动处理（写正则或者用其他的字符串函数），或者手动找一些能便于处理的库（MDN文档中提供了一个示例）。

那么还有一个新的问题：如何删除一个Cookie？还记得之前提到的`Max-Age`吗，只要设置`expire`或者`Max-age`为0或者1970就可以。例如下面的代码：

```js
// 删除 Cookie
document.cookie = "username=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;";
```






## 参考资料

+ [HTTP Cookie - HTTP | MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/Cookies)
+ [Document.cookie - Web API | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/cookie)