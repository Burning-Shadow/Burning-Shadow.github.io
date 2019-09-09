---
title: JavaScript中的call和apply
categories: JavaScript
---

`call`和`apply`均用于`this`值绑定，区别在于参数列表
>`Function.apply(thisObj, args)`
>`Function.call(thisObj, [param1[, param2 [, param3 [... [,paramN]]]]])`

前者传入参数列表（`arguments`），后者挨个向里边放参数

<!--more-->

`call`、`apply`方法可以将一个函数的对象上下文从出事的上下文改编为由`thisObj`指定的新对象


此章作为[this指向](https://blog.csdn.net/qq_38722097/article/details/88125450)章节的补充
![在这里插入图片描述](https://ww1.sinaimg.cn/large/007i4MEmgy1g1avrg3n5gj30kq0kqq3j.jpg)