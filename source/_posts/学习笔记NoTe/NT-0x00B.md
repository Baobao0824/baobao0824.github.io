---
title: Understanding the purpose of *args and **kwargs in Python - NoTe0x00B
date: 2026-06-29 14:09:26
tags:
---


> 原链接：
> https://www.pythonsnacks.com/p/understanding-the-purpose-of-args-and-kwargs-in-python

## 写在前面

这一系列是我在学习他人的优质课程（博客文章、视频课程等）时所做的学习笔记，根据我自身的水平来进行学习，同时会进行思维的发散，补充原文中没有提到或者有错误的地方。同时也会进行经常性的更新和整理，让它和我的当下的状态更加契合。

整了那么多大的，咱再来整一点小的。聚焦一下python的一点小八股和语法。

## 正文

首先，我们要了解什么是位置参数（positional arguments）和关键词参数（keyword arguments）。例如：

```py
func(phrase = "hello")
# 这是一个关键词参数
func("hello", phrase="world")
# 第一个这种普通的、根据位置来判定的参数就是未知参数，后面这个phrase参数就是关键词参数
```

首先我们要了解`*`和`**`这两个运算符到底是什么。他们都是前缀运算符，用来捕获传递到函数中的参数。例如：

```py
numbers = [7,3,6,2,5,1,4]
more_numbers = [-5,*numbers,-10]
print(more_numbers)
# [-5, 7, 3, 6, 2, 5, 1, 4, -10]
print(*more_numbers)
```

如果使用`*more_numbers`，就是把列表中的所有项作为独立参数传入到函数中，函数甚至不知道列表中有多少参数。这不仅是语法糖，如果没有这个运算符，你都不知道在列表长度不固定的情况下如何把所有参数直接传过去。

```py
print(*numbers)
# 7 3 6 2 5 1 4
print(numbers[0],numbers[1],numbers[2])
# 7 3 6
```

类似的，`**`运算符也有这个功能，但是它最主要的是允许我们将一个键值对的字典能够拆包成函数调用的关键词参数。

```py
date_info={
    'year': '2026',
    'month': '6',
    'day': '26'
}
def func(year, month,day):
    print(month,day,year)
    pass

func(**date_info)
# 6 26 2026
```

甚至我们还可以在函数中多次使用。

```py
numbers = [1,2,3,4]
alphabet = ['a','b','c']
print(*numbers,*alphabet)
# 1 2 3 4 a b c
date_info={
    'year': '2026',
    'month': '6',
    'day': '26'
}
people = {
    'name': 'baobao',
    'age': 22
}
def func(year, month,day,name,age):
    print(month,day,year,name,age)
    pass
func(**date_info,**people)
# 6 26 2026 baobao 22
def func(year, month,day):
    print(month,day,year)
    pass
func(**date_info,**people)
# TypeError: func() got an unexpected keyword argument 'name'
def func(year, month,day,name,age):
    print(month,day,year,name,age)
    pass
numbers=[1,2,3,4,5]
func(*numbers)
# 2 3 1 4 5
```

注意上面的错误了吗，如果你给的参数名称超过了参数列表中的名称，那么就是传入了一个多余的参数，python就会报错。同时，由于形参名称不能重复，你也要保证你传入的所有字典键值没有重复的，否则也会报错。

```py
date_info={
    'year': '2026',
    'month': '6',
    'day': '26'
}
people = {
    'name': 'baobao',
    'age': 22,
    'day':'2'
}
def func(year, month,day,name,age):
    print(month,day,year,name,age)
    pass
func(**date_info,**people)
# TypeError: __main__.func() got multiple values for keyword argument 'day'
```

但是除此之外，我们在很多的著名第三方库中，都能看到这样的用法。

```py
def function (*args, **kwargs):
    print(args)
    print(kwargs)
```

可以看到这里有两个关键的参数，是`args`和`kwargs`。首先我们来讨论`*args`，在定义函数是可以用它捕获无限个位置参数，然后就能将其捕获到一个元组中（当然不叫`args`也是可以的）。

```py
def func(*args):
    for item in args:
        print(item)

func(1,2,43,4,5)
# 1
# 2
# 43
# 4
# 5
func(1)
# 1
```

接下来我们再说`**kwargs`，我们可以通过在函数定义中用这个运算符来捕捉函数中给出的任何关键字参数到一个字典中，也就是说，函数的`kwargs`是一个字典，里面包含着你传入的键值对。

```py
def func(**kwargs):
    for lower, upper in kwargs.items():
        print(lower,upper,sep='')

func(a='A',b='B')
# aA
# bB
```

如果我们全都用关键词参数，那确实很爽，但是有的时候我们不得不面对未知的无限个未知参数（例如标准库里面的`print`函数，它首先接收的是`*values`，然后是`sep`）这个时候我们应该怎么办呢？这个时候我们可以单独用一个`*`来表示。

```py
def func(array,*,sep='-'):
    for i in array:
        print(i,sep)

func([1,2,3,4],sep='a')
# 1 a
# 2 a
# 3 a
# 4 a
func([1,2,3,4],'a')
# TypeError: func() takes 1 positional argument but 2 were given
```

这里的`sep`必须指定为关键字参数，`*`的作用就是让解释器了解后面不再有未知参数了，全都是关键字参数。python内部的排序`sorted`函数实际上就采用了这种方法。

```py
help(sorted)
# sorted(iterable, /, *, key=None, reverse=False)
#     Return a new list containing all items from the iterable in ascending order.
```

除了这些用法之外，`*`还可以直接用来给tuple或者list拆包


```py
alphabet=['a','b','c','d']
first, second,*last = alphabet
first, second,last
# ('a', 'b', ['c', 'd'])
first,*middle,last = alphabet
first,middle,last
# ('a', ['b', 'c'], 'd')
```

你也可以连续用多个，但是作者说他没见过有什么好的用法，看起来也很晦涩难懂，所以不推荐我们使用。

在python3.5之后，我们也可以通过`*`将迭代数据导入新列表。例如我们原本有这样一个函数：

```py
def func(seq):
    return list(seq) + list(reversed(seq))

func(['a','b','c'])
# ['a', 'b', 'c', 'c', 'b', 'a']
```

这样看上去很麻烦，有没有什么更简单的方法呢？您好有的。

```py
def func(seq):
    return [*seq, *reversed(seq)]

func(['a','b','c'])
# ['a', 'b', 'c', 'c', 'b', 'a']
```

我们还可以写一些更花里胡哨的进阶用法。例如我们写一个函数，接收一个列表，把列表的第一个元素移到末尾。

```py
def func (a:list):
    return [*a[1:],a[0]]

func([1,2,3,4])
# [2, 3, 4, 1]
```

那么`**`有什么进阶用法吗，也是有的，在python3.5之后，我们可以将一个字典中的键值对转入到新的字典中。甚至两个字典中有重复的键名也不会报错，后一个字典中的值会覆盖前一个。

```py
date_info_1 ={
    'year': '2026',
}
date_info_2 = {
    'month': '6',
    'day': '29'
}
date_info ={
    **date_info_1,
    **date_info_2
}
date_info
# {'year': '2026', 'month': '6', 'day': '29'}
date_info_1 ={
    'day': '2026',
}
date_info_2 = {
    'month': '6',
    'day': '29'
}
date_info ={
    **date_info_1,
    **date_info_2
}
date_info
# {'day': '29', 'month': '6'}
```

## 总结

python中的`*`和`**`虽然没有正式名称，并且用法多样，但是他们的用途核心都是一样的，`*`用于将列表解包成多个变量，而`**`用于将字典解包为键值对，还是很好用的，可以在写代码的时候多使用当做很好的语法糖，就是不要写的太奇怪了让人看不懂就好。

## 参考资料

+ [Asterisks in Python: what they are and how to use them](https://treyhunner.com/2018/10/asterisks-in-python-what-they-are-and-how-to-use-them/)