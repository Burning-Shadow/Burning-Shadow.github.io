---
title: ES6中的 Generator
date: 2019-04-06 10:13:00
categories: ES6
tags: 
- generator
- function*
- yield
- iterator
- yield*
---

### 前言

今天讨论的新特性让我非常兴奋，因为这个特性是 ES6 中最神奇的特性。

这里的“神奇”意味着什么呢？对于初学者来说，该特性与以往的 JS 完全不同，甚至有些晦涩难懂。从某种意义上说，它完全改变了这门语言的通常行为，这不是“神奇”是什么呢。

不仅如此，该特性还可以简化程序代码，将复杂的“回调堆栈”改成线性执行的形式。

<!--more-->

### 简介

- 通常的函数以 `function` 创建生成器，但 Generator 函数以 `function*` 创建。
- 在 Generator 函数内部，`yield` 是一个关键字，和 `return` 有点像。不同点在于，所有函数（包括 Generator 函数）都只能返回一次，而在 Generator 函数中可以 yield 任意次。*yield 表达式暂停了 Generator 函数的执行，然后可以从暂停的地方恢复执行。*

常见的函数不能暂停执行，而 Generator 函数可以，这就是这两者最大的区别。

#### Iterator

迭代器（`Iterator`）表示可以被遍历迭代的对象，以编程的方式返回集合中的下一项，迭代器对象都拥有`next()`方法，返回`{value:'',done:bool}`，`value`表示当前迭代位置的返回值，`done`表示是否迭代结束，结束时其值为`false`。

迭代对象之所以可被迭代是因为其有`Symbol(Symbol.iterator)`属性。如果有什么疑问可以参考上一章。

#### Generator

生成器（`Generator`）是一个返回迭代器对象（`Iterator`）的函数

- 使用`function* funName(){}`创建
- 使用声称其表达式创建生成器：`let iterator = function* (){}`
- 函数体中使用`yield`关键字定义每次迭代器调用`next()`返回的`value`结果。`yield`指令相当于起到暂停代码执行的作用。只有在迭代器调用`next()`时，才会出发下一个`yield`继续执行
- 迭代完`yield`后，下一次迭代会返回`return`的值。如果`return`在中间，则迭代到`return`行后直接跳出。

```javascript 
function *createIterator(){
    yield 1;
    yield 2;
    yield 3;
    return 123
}
let iterator = createIterator()
console.log(iterator.next()) //{value: 1, done: false}
console.log(iterator.next()) //{value: 2, done: false}
console.log(iterator.next()) //{value: 3, done: false}
console.log(iterator.next()) //{value: 123, done: true}
console.log(iterator.next()) //{value: undefined, done: true}
```

### 原理

当我们调用一个`Generator`函数时并没有立即执行，而是返回了一个`generator`对象，也就是上边的`iterator`

。此时函数就立即暂停在函数代码的第一个`yield`处。

当我们每次调用`Generator`对象的`.next()`方法时，函数就开始执行，直至遇到下一个`yield`表达式为止。



所以我们每次调用`iterator.next()`时都会得到一个不同的字符串，这些字符串都是在函数内部通过`yield`表达式产生的值。



从技术层面上讲，每当`Generator`函数执行遇到`yield`表达式时，函数的栈帧 – 本地变量，函数参数，临时值和当前执行的位置，就从堆栈移除，但是`Generator`对象保留了对该栈帧的引用，所以下次调用`.next()`方法时，就可以恢复并继续执行。

值得提醒的是`Generator`并不是多线程。在支持多线程的语言中，同一时间可以执行多段代码，并伴随着执行资源的竞争，执行结果的不确定性和较好的性能。而`Generator`函数并不是这样，当一个`Generator`函数执行时，它与其调用者都在同一线程中执行，每次执行顺序都是确定的，有序的，并且执行顺序不会发生改变。与线程不同，`Generator`函数可以在内部的`yield`的标志点暂停执行。

### 迭代器 Iterator

通过上篇文章，我们知道**迭代器并不是 ES6 的一个内置的类，而只是作为语言的一个扩展点**，你可以通过实现 `[Symbol.iterator]()` 和 `.next()` 方法来定义一个迭代器。

但是，实现一个接口还是需要写一些代码的，下面我们来看看在实际中如何实现一个迭代器，以实现一个 `range` 迭代器为例，该迭代器只是简单地从一个数累加到另一个数，有点像 C 语言中的 `for (;;)` 循环。

```javascript
// This should "ding" three times
for (var value of range(0, 3)) {
  alert("Ding! at floor #" + value);
}
```

现在有一个解决方案，就是使用 ES6 的类。（如果你对 `class` 语法还不熟悉，不要紧，我会在将来的文章中介绍。）

```javascript
class RangeIterator {
  constructor(start, stop) {
    this.value = start;
    this.stop = stop;
  }
  [Symbol.iterator]() { return this; }
  next() {
    var value = this.value;
    if (value < this.stop) {
      this.value++;
      return {done: false, value: value};
    } else {
      return {done: true, value: undefined};
    }
  }
}
// Return a new iterator that counts up from 'start' to 'stop'.
function range(start, stop) {
  return new RangeIterator(start, stop);
}
```

这种实现方式与 [Java](http://gafter.blogspot.com/2007/07/internal-versus-external-iterators.html) 和 [Swift](https://schani.wordpress.com/2014/06/06/generators-in-swift/) 的实现方式类似，看上去还不错，但还不能说上面代码就完全正确，代码没有任何 Bug？这很难说。我们看不到任何传统的 `for (;;)` 循环代码：迭代器的协议迫使我们将循环拆散了。



不过我们如果通过`Generator`函数实现迭代器那么就会容易很多

```javascript
function* range(start, stop) {
  for (var i = start; i < stop; i++)
    yield i;
}
```

上面这 4 行代码就可以完全替代之前的那个 23 行的实现，替换掉整个 `RangeIterator` 类，这是因为 Generator 天生就是迭代器，所有的 Generator 都原生实现了 `.next()` 和 `[Symbol.iterator]()` 方法。你只需要实现其中的循环逻辑就够了。



我们可以使用作为迭代器的 Generator 的哪些功能呢？

- **使任何对象可遍历** – 编写一个 Genetator 函数去遍历 `this`，每遍历到一个值就 yield 一下，然后将该 Generator 函数作为要遍历的对象上的 `[Symbol.iterator]` 方法的实现。
- **简化返回数组的函数** – 假如有一个每次调用时都返回一个数组的函数，比如：

```javascript
// Divide the one-dimensional array 'icons'
// into arrays of length 'rowLength'.
function splitIntoRows(icons, rowLength) {
  var rows = [];
  for (var i = 0; i < icons.length; i += rowLength) {
    rows.push(icons.slice(i, i + rowLength));
  }
  return rows;
}
```

使用`Generator`可以简化这类函数

```javascript
function* splitIntoRows(icons, rowLength) {
  for (var i = 0; i < icons.length; i += rowLength) {
    yield icons.slice(i, i + rowLength);
  }
}
```

这两者唯一的区别在于，前者在调用时计算出了所有结果并用一个数组返回，后者返回的是一个迭代器，结果是在需要的时候才进行计算，然后一个一个地返回。

- **无穷大的结果集** – 我们不能构建一个无穷大的数组，但是我们可以返回一个生成无尽序列的 Generator，并且每个调用者都可以从中获取到任意多个需要的值。
- **重构复杂的循环** – 你是否想将一个复杂冗长的函数重构为两个简单的函数？Generator 是你重构工具箱中一把新的瑞士军刀。对于一个复杂的循环，我们可以将生成数据集那部分代码重构为一个 Generator 函数，然后用 `for-of` 遍历：`for (var data of myNewGenerator(args))`。
- **构建迭代器的工具** – ES6 并没有提供一个可扩展的库，来对数据集进行 `filter` 和 `map` 等操作，但 Generator 可以用几行代码就实现这类功能。

例如，假设你需要在`Nodelist`上实现与 `Array.prototype.filter` 同样的功能的方法。小菜一碟的事：

```javascript
function* filter(test, iterable) {
  for (var item of iterable) {
    if (test(item))
      yield item;
  }
}
```

所以，Generator 很实用吧？当然，这是实现自定义迭代器最简单直接的方式，并且，在 ES6 中，迭代器是数据集和循环的新标准。

但，这还不是 Generator 的全部功能。

### 生成器委托 yield*

```javascript
function* g1() {
  yield 1;
  yield 2;
}

function* g2() {
  yield* g1();
  yield* [3, 4];
  yield* "56";
  yield* arguments;
}

var generator = g2(7, 8);
console.log(...generator);   // 1 2 3 4 "5" "6" 7 8
```

这玩意儿其实也很好理解，带*的函数是我们的`generator`语句，而`yield`开头的则是我们的“产出”语句。二者相结合可以理解为执行至`yield`时继续创建一个`generator`，emmmm。。。很容易理解吧~

没理解我暂时也没想到有什么解释的方法（小声bb。。。）

### 分段式代码

当我们需要手动控制异步进程时，可以通过将`Promise`和`Generator`结合起来实现

```javascript
// 异步ajax代码
var getJSON = (url)=>{
    var promise = new Promise((resolve, reject)=>{
        var client = new XMLHttpRequest()
        client.open('GET', url)
        client.onreadystatechange = handler
        client.responseType = 'json'
        client.setRequestHeader('Accept', 'application/json')
        client.send()
        
        function handler(){
            if(this.readyState !== 4){
                return;
            }
            if(this.status === 200){
                resolve(this.response)
            }else{
                reject(new Error(this.statusText))
            }
        }
    })
    return promise
}

// 迭代器
function* fun(){
    yield (a = getJSON('http://www.baidu.com'))
    // 这里你可以干一些其他事情
    yield a.then(json => {
    	this.$refs.dom.src = json.src
    })
}

let a = fun()
a.next()	// 发送异步请求
a.next()	// 更新DOM
```

### 兼容性

在服务器端，现在就可以直接在 io.js 中使用 `Generator`（或者在 NodeJs 中以 `--harmony` 启动参数来启动 Node）。

在浏览器端，目前只有 Firefox 27 和 Chrome 39 以上的版本才支持 Generator，如果想直接在 Web 上使用，你可以使用 [Babel](http://babeljs.io/) 或 Google 的 [Traceur](https://github.com/google/traceur-compiler#what-is-traceur) 将 ES6 代码转换为 Web 友好的 ES5 代码。

一些题外话：JS 版本的 Generator 最早是由`Brendan Eich`实现，他借鉴了 [Python Generator](https://www.python.org/dev/peps/pep-0255/) 的实现，该实现的灵感来自 [Icon](http://www.cs.arizona.edu/icon/)，早在 [2006 年](https://developer.mozilla.org/en-US/docs/Web/JavaScript/New_in_JavaScript/1.7)的 Firefox 2.0 就吸纳了 Generator。但标准化的道路是坎坷的，一路下来，其语法和行为都发生了很多改变，Firefox 和 Chrome 中的 ES6 Generator 是由 [Andy Wingo](http://wingolog.org/) 实现 ，这项工作是由 Bloomberg 赞助的。



### 实现

```javascript
// cb 也就是编译过的 test 函数
function generator(cb) {
  return (function() {
    var object = {
      next: 0,
      stop: function() {}
    };

    return {
      next: function() {
        var ret = cb(object);
        if (ret === undefined) return { value: undefined, done: true };
        return {
          value: ret,
          done: false
        };
      }
    };
  })();
}
// 如果你使用 babel 编译后可以发现 test 函数变成了这样
function test() {
  var a;
  return generator(function(_context) {
    while (1) {
      switch ((_context.prev = _context.next)) {
        // 可以发现通过 yield 将代码分割成几块
        // 每次执行 next 函数就执行一块代码
        // 并且表明下次需要执行哪块代码
        case 0:
          a = 1 + 2;
          _context.next = 4;
          return 2;
        case 4:
          _context.next = 6;
          return 3;
		// 执行完毕
        case 6:
        case "end":
          return _context.stop();
      }
    }
  });
}
```







