---
title: Generator+Promise实现Async&Await
date: 2019-04-16 00:29:00
categories: ES6
tags:
 - Async
 - Await
 - Generator
 - Promise
---

我们在之前讲解`async`时提到了由`Promise`转变至`async`所经历的一系列过程。

**由于`Promise`只能通过`catch`捕获错误，而在内部`.then`链中嵌套的一系列`Promise`调用所产生的`err`是无法被外层`catch`所捕获的，而其后添加`catch`则又破坏了代码的格式和一致性。**

**同时，`Promise`中的`catch`函数中的异常堆栈不够完整，依旧难以追寻真正发生错误的位置（因为内部调用大都是匿名函数。而不用匿名函数则又丧失了箭头函数的简洁，让开发者很苦恼）。**

所以，`Async`语法应运而生。他弥补了上述`Promise`调用所出现的不足之处，而其具体实现则是通过`Generator`和`Promise`协同的语法糖。下面我们一起看看如何通过`Generator`和`Promise`实现一个`Async-Await`吧~

<!--more-->

### Generator

- `Generator`是一个函数，通过`yield`关键字控制其执行
- 执行该函数时到第一个`yield`时停止执行，直到调用其返回值的`.next()`才会执行`yield`后面的部分，返回一个带有`value`和`done`属性的对象，**其机制类似于`return`，到此为止，返回结果**。
- 我们可以在调用`next()`时传递一个参数，在上次`yield`前接收到这个参数
- 同时，`Generator`内部仍可以嵌套`Generator`函数，具体实现就是`yield*`，在此不再赘述。



在此只对其做一个粗浅的概述，若各位有兴趣可以看相关的博客： [《ES6中的Generator》](https://burning-shadow.github.io/2019/03/24/ES6%E4%B8%AD%E7%9A%84-Generator/)

### Promise

- 每次返回更新自己的状态（`fullfied` || `rejected`），返回值仍为一个`Promise`

### 实现

实现之前我们需要明确：

- `generator`生成器返回值是一个对象（`{value: XXX, done: true}`）
- `async-await`返回的是一个`Promise`。

所以我们若想实现功能，必须将`Promise`塞进`field`之中去

```javascript
function run (gen) {
  gen = gen()
  return next(gen.next())	// 自动调用yeild

  function next ({done, value}) {
    return new Promise(resolve => {	// 返回Promise，与async返回类型相同
     if (done) { // finish
       resolve(value)
     } else { // not yet
       value.then(data => {
         next(gen.next(data)).then(resolve)	// 递归调用yield
       })
     }
   })
  }
}

function getRandom () {
  return new Promise(resolve => {
    setTimeout(_ => resolve(Math.random() * 10 | 0), 1000)
  })
}

function * main () {
  let num1 = yield getRandom()
  let num2 = yield getRandom()

  return num1 + num2
}

run(main).then(data => {
  console.log(`got data: ${data}`);
})
```

由代码可知我们通过`generator`——`main`函数以近似同步的方式执行异步代码。但这同时也需要一个外部函数帮助我们执行这个`main`。



而其对应的`async/await`函数实现上方代码如下

```javascript
function getRandom () {
  return new Promise(resolve => {
    setTimeout(_ => resolve(Math.random() * 10 | 0), 1000)
  })
}

async function main () {
  let num1 = await getRandom()
  let num2 = await getRandom()

  return num1 + num2
}

console.log(`got data: ${await main()}`)
```

大体上看我们只需要将`*`改为`async`，将`yield`改成`await`，且免去了外部调用的`main`函数。



事实上`Generator`作为一个生成器，用以完成异步代码的近同步化确实有些不务正业。不过的确有相关的库（`co`）用以将`Promise`和`Generator`实现其功能，有兴趣的`boy`可以查询相关文档。

### 总结

`Generator`生成器返回的对象是一个类似于`{value: XXX, done: true}`结构的`Object`。而`Async`则始终返回一个`Promise`，使用`await`或`.then()`获取返回值。



相较于`Generator + co`，`Async`生来就是为了处理异步编程，对此更加专业，所以还是让`Generator`干他自己的老本行去8~

















