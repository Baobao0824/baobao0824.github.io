---
title: 在python项目中使用yaml作为配置文件的推荐方式
date: 2025-12-25 15:15:25
tags: [代码, python]
categories: 代码
---

## 前言

众所周知，目前市面上有几个比较主流的、用来做配置文件的标记语言。在js环境中，用json比较多；在java环境中主要用的是xml，而python环境中用的则是yaml。由于组里的项目在要求我们“能通过yaml文件来自定义配置”，因此就顺便学了一下yaml的使用，同时也摸索了一下比较好用的使用方式。虽然不一定可能是python官方推荐的最佳实践，但是确实在平常写代码的时候比较好用。

## 教程正文

### yaml语言的结构

yaml是一个标记语言，我们来看菜鸟教程的介绍：

> YAML 是 "YAML Ain't a Markup Language"（YAML 不是一种标记语言）的递归缩写。在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言）。
> 
> YAML 的语法和其他高级语言类似，并且可以简单表达清单、散列表，标量等数据形态。它使用空白符号缩进和大量依赖外观的特色，特别适合用来表达或编辑数据结构、各种配置文件、倾印调试内容、文件大纲（例如：许多电子邮件标题格式和YAML非常接近）。

这种话其实对平常写过代码的人来说没什么用处。所以我会用我认为比较快的方式直接给大家直观的展示yaml：

```json
// 一个用来示例的json结构
{
    "foo": "bar",
    "a": 100,
    "b": {
        "c": "c",
        "arr": [
            "str0",
            "str1"
        ]
    }
}
```

我们把这个结构转换成yaml，就是下面这个样子：

```yaml
# 这个json对应的yaml
foo: 'bar'
a: 100
b:
    c: "c"
    arr: [
        "str0",
        "str1"]
```

[Yaml Checker](https://yamlchecker.com/)这个网站可以查看你输入的yaml是否合法。

### 用python读取yaml文件

直接在你的项目中新建一个yaml文件作为你的配置文件，例如`config.yaml`，作为教程，内容就直接用上面的复制过来就好了。那么如何在本地读取呢？我们需要一个第三方库（是的，主要在python中用的文件格式，python标准库中居然没有），它的pip包名叫做`pyyaml`。可以在任何支持的包管理器和虚拟环境中进行安装。不过注意，在真正的python文件中，我们不能直接用`pyyaml`这个名称，而是直接`import yaml`就好。下面是一份示例的读取代码。

```py
# config_loader.py
from pathlib import Path
import yaml

PROJECT_ROOT = Path()# 你的项目根目录，一个Path类别的变量
CONFIG_PATH = PROJECT_ROOT / "config.yaml" #配置文件名

# 读取文件
with CONFIG_PATH.open("r", encoding="utf-8") as f:
    CONFIG = yaml.safe_load(f)
```

接下来，在你需要引用配置的地方，引用`CONFIG`这个变量就可以了，例如`CONFIG['b']['c']`。

### 如何实现代码补全

如果直接这样使用的话，那么我们是很难在vscode或者其他IDE中得到补全或者提示之类的高级功能的，如果想要这些的话，我们必须要让`CONFIG`继承自`TypedDict`类，这样就可以做到直接使用类型补全了。

```py
# config_dict.py
from typing import TypedDict

# 只要出现了嵌套，就必须要再注册一个类
# 前面加上_就代表是一个内部类
class _B(TypedDict):
    c: str
    arr: list

class ConfigDict(TypedDict):
    foo: str
    a: int
    b: _B
```

对应的，在`config_loader`中，也要指定`CONFIG`的类型为`ConfigDict`。这样的话，在vscode中就可以直接使用代码补全了。

![就是这样的效果](1.png)

## Q&A

> 为什么要用yaml，它相比于json的优势在哪里？

Kimi给出的回答，整合之后答案如下：

+ 人类可读性强，毕竟少了很多大括号和引号，而且数据通过缩进展示，更加直观
+ 原生支持注释，而正儿八经的原版JSON规范其实按理说是不支持注释的（虽然很多人都会写）
+ 类型更多，原生支持bool、日期等类型
+ 支持锚点和引用，可以避免重复配置

你让我总结，那我的总结就是yaml比json功能更多，更加高级。

> 什么是TypedDict，为什么要用它？

Python3.8起，标准库`typing`引入了这个类型，本质上是对一个`dict`的加强版。让你在运行的时候，虽然拿到的是普通的dict，但是能够给键名和值的类型都加上静态检查，这样就可以把一个`dict`当成类似于C语言的`struct`来用。至于为什么是基于`dict`，因为如果你直接写类的话，很多场景（例如使用JSON）就不是很方便用了。而普通的`dict`又太松散了，类似于JavaScript中的`Object`，很难起到任何的类型检查的作用，成为了滋生bug的温床。因此使用TypedDict，成为了既想要灵活，又想要类型提示的一个简单方案。

> 为什么包名叫pyyaml，使用的时候引入的却是yaml？

因为包名不等于import的名字，pyyaml这个包的代码根目录名字就叫yaml。类似的还有`Pillow`（引入的是`PIL`）、`BeautifulSoup4`（引入的是`bs4`）等。

## 总结

之前在组里的项目中就有这一个要求，因此就直接写下来，以后在写其他的python项目的时候，也可以把这个最佳实践直接拿来用，这样的话就可以很好的去耦合，不用再天天去文件里改常量了。

## 更新记录

+ 2025.12.25 发布第一稿