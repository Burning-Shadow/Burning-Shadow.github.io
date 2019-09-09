---
title: ES6 中的 for-of 和 Iterator
categories: ES6
tags: 
- Iterator
- for...of
---

### 前言

#### 先从遍历数组开始

在刚上手`js`的时候如果让你遍历数组你可能会像玩C语言一样`for(var index = 0; index < myArray.length; index++)`。而从`ES5`之后我们可以使用内置的`forEach`方法，大大提升了代码的可读性（并没有）。

**而`forEach`有一个弊端，就是无法使用`break`跳出循环，也不能用`return`语句从闭包函数中返回**

<!--more-->

#### 那么 for-in 怎样

```javascript
for (var index in myArray) { // 实际代码中不要这么做
  console.log(myArray[index]);
}
```

这样不好，因为：

- **上面代码中的 `index` 变量将会是 `"0"`、`"1"`、`"3"` 等这样的字符串，而并不是数值类型**。如果你使用字符串的 `index` 去参与某些运算（`"2" + 1 == "21"`），运算结果可能会不符合预期。
- **不仅数组本身的元素将被遍历到，那些由用户添加的[附加（expando）元素](https://developer.mozilla.org/en-US/docs/Glossary/Expando)也将被遍历到**，例如某数组有这样一个属性 `myArray.name`，那么在某次循环中将会出现 `index="name"` 的情况。而且，**甚至连数组原型链上的属性也可能被遍历到**。
- 最不可思议的是，**在某些情况下，上面代码将会以任意顺序去遍历数组元素**。

简单来说，`for-in` 设计的目的是用于遍历包含键值对的对象，对数组并不是那么友好。

> `for-in`被设计出来的初衷就是遍历对象的属性，所以才可能会波及元素的**属性**、**原型链**、

#### 引入for-of

- `for-in`用于遍历对象的属性
- `for-of`用于遍历数据 - 就像数组中的元素

用这东西执行遍历操作显然好得多啦

#### 适用方向

##### 数组

这个咱们就不多说啦

##### 类数组的对象

比如 DOM 对象的集合`NodeList`

##### 遍历字符串

它将字符串看作是`Unicode`字符的集合

```javascript
for (var chr of "😺😲") {
  alert(chr);
}
```

##### `Map`和`Set`对象

- ```javascript
  var uniqueWords = new Set(words);
  for (var word of uniqueWords) {
    console.log(word);
  }
  ```

- ```javascript
  for (var [key, value] of phoneBookMap) {
    console.log(key + "'s phone number is: " + value);
  }
  ```

### for-of 内部原理

> “好的艺术家复制，伟大的艺术家偷窃” —— 巴伯罗.毕加索

被添加到 ES6 中的那些新特性并不是无章可循，大多数特性都已经被使用在其他语言中，而且事实也证明这些特性很有用。

就拿 `for-of` 语句来说，在 C++、JAVA、C# 和 Python 中都存在类似的循环语句，并且用于遍历这门语言和其标准库中的各种数据结构。

与其他语言中的 `for` 和 `foreach` 语句一样，`for-of` **要求被遍历的对象实现特定的方法**。**所有的 `Array`、`Map` 和 `Set` 对象都有一个共性，那就是他们都实现了一个迭代器（iterator）方法。**



这就像你可以为一个对象实现一个 `myObject.toString（）` 方法，来告知 JS 引擎如何将一个对象转换为字符串；你也可以为任何对象实现一个 `myObject[Symbol.iterator]()` 方法，来告知 JS 引擎如何去遍历该对象。

```javascript
jQuery.prototype[Symbol.iterator] = Array.prototype[Symbol.iterator];
```

你也许在想，为什么 `[Symbol.iterator]` 语法看起来如此奇怪？这句话到底是什么意思？问题的关键在于方法名，ES 标准委员会完全可以将该方法命名为 `iterator()`，但是，现有对象中可能已经存在名为“iterator”的方法，这将导致代码混乱，违背了最大兼容性原则。所以，标准委员会引入了 `Symbol`，而不仅仅是一个字符串，来作为方法名。

标准委员会引入全新的 `Symbol`，比如 `Symbol.iterator`，是为了不与之前的代码冲突。唯一不足就是语法有点奇怪，但对于这个强大的新特性和完美的后向兼容来说，这个就显得微不足道了。

一个拥有 `[Symbol.iterator]()` 方法的对象被认为是可遍历的（`iterable`）。在后面的文章中，我们将看到“可遍历对象”的概念贯穿在整个语言中，不仅在 `for-of` 语句中，而且在 `Map` 和 `Set` 的构造函数和析构（`Destructuring`）函数中，以及新的扩展操作符中，都将涉及到。

### 迭代器对象 Iterator

#### Iterator 是什么

遍历器（`Iterator`）就是一种统一的接口机制，用以处理不同的数据结构

它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署`Iterator`接口，就可以完成遍历操作（即依次处理该数据结构的所有成员）。



`Iterator`的作用有三个：

- 一是为各种数据结构，提供一个统一的、简便的访问接口；
- 二是使得数据结构的成员能够按某种次序排列；
- 三是 ES6 创造了一种新的遍历命令`for...of`循环，Iterator 接口主要供`for...of`消费。

#### 具备 Iterator 接口的数据结构

- `Array`
- `Map`
- `Set`
- `String`
- `TypedArray`
- 函数的`arguments`对象
- `NodeList`对象

```javascript
let arr = ['a', 'b', 'c'];
let iter = arr[Symbol.iterator]();

iter.next() // { value: 'a', done: false }
iter.next() // { value: 'b', done: false }
iter.next() // { value: 'c', done: false }
iter.next() // { value: undefined, done: true }
```



对象（Object）之所以没有默认部署 Iterator 接口，是因为对象的哪个属性先遍历，哪个属性后遍历是不确定的，需要开发者手动指定。**本质上，遍历器是一种线性处理，对于任何非线性的数据结构，部署遍历器接口，就等于部署一种线性转换**。不过，严格地说，对象部署遍历器接口并不是很必要，因为这时对象实际上被当作 Map 结构使用，ES5 没有 Map 结构，而 ES6 原生提供了。

**一个对象如果要具备可被`for...of`循环调用的 `Iterator` 接口，就必须在`Symbol.iterator`的属性上部署遍历器生成方法（原型链上的对象具有该方法也可）。**

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
    }
    return {done: true, value: undefined};
  }
}

function range(start, stop) {
  return new RangeIterator(start, stop);
}

for (var value of range(0, 3)) {
  console.log(value); // 0, 1, 2
}
```

上面代码是一个类部署 `Iterator` 接口的写法。`Symbol.iterator`属性对应一个函数，执行后返回当前对象的遍历器对象。

#### Iterator 怎么用

通常我们不会完完全全从头开始去实现一个迭代器（Iterator）对象，下一篇文章将告诉你为什么。但为了完整起见，让我们来看看一个迭代器对象具体是什么样的。（如果你跳过了本节，你将会错失某些技术细节。）

就拿 `for-of` 语句来说，它首先调用被遍历集合对象的 `[Symbol.iterator]()` 方法，该方法返回一个迭代器对象，迭代器对象可以是拥有 `.next` 方法的任何对象；然后，在 `for-of` 的每次循环中，都将调用该迭代器对象上的 `.next` 方法。下面是一个最简单的迭代器对象：

```javascript
var zeroesForeverIterator = {
  [Symbol.iterator]: function () {
    return this;
  },
  next: function () {
    return {done: false, value: 0};
  }
};
```

在上面代码中，每次调用 `.next()` 方法时都返回了同一个结果，该结果一方面告知 `for-of`语句循环遍历还没有结束，另一方面告知 `for-of` 语句本次循环的值为 `0`。这意味着 `for (value of zeroesForeverIterator) {}` 是一个死循环。当然，一个典型的迭代器不会如此简单。

ES6 的迭代器通过 `.done` 和 `.value` 这两个属性来标识每次的遍历结果，这就是迭代器的设计原理，这与其他语言中的迭代器有所不同。在 Java 中，迭代器对象要分别使用 `.hasNext()`和 `.next()` 两个方法。在 Python 中，迭代器对象只有一个 `.next()` 方法，当没有可遍历的元素时将抛出一个 `StopIteration` 异常。但从根本上说，这三种设计都返回了相同的信息。

迭代器对象可以还可以选择性地实现 `.return()` 和 `.throw(exc)` 这两个方法。如果由于异常或使用 `break` 和 `return` 操作符导致循环提早退出，那么迭代器的 `.return()` 方法将被调用，可以通过实现 `.return()` 方法来释放迭代器对象所占用的资源，但大多数迭代器都不需要实现这个方法。`throw(exc)` 更是一个特例：在遍历过程中该方法永远都不会被调用，关于这个方法，我会在下一篇文章详细介绍。

现在我们知道了 `for-of` 的所有细节，那么我们可以简单地重写该语句。

首先是 `for-of` 循环体：

```javascript
for (VAR of ITERABLE) {
  STATEMENTS
}
```

这只是一个语义化的实现，使用了一些底层方法和几个临时变量

```javascript
var $iterator = ITERABLE[Symbol.iterator]();
var $result = $iterator.next();
while (!$result.done) {
  VAR = $result.value;
  STATEMENTS
  $result = $iterator.next();
}
```

上面代码并没有涉及到如何调用 `.return()` 方法，我们可以添加相应的处理，但我认为这样会影响我们对内部原理的理解。`for-of` 语句使用起来非常简单，但在其内部有非常多的细节。

#### 用到 Iterator 接口的场合

##### 解构赋值

对数组和 Set 结构进行解构赋值时，会默认调用`Symbol.iterator`方法。

```javascript
let set = new Set().add('a').add('b').add('c');

let [x,y] = set;
// x='a'; y='b'

let [first, ...rest] = set;
// first='a'; rest=['b','c'];
```

##### 扩展运算符

扩展运算符（`...`）也会调用默认的 `Iterator` 接口。

```javascript
// 例一
var str = 'hello';
[...str] //  ['h','e','l','l','o']

// 例二
let arr = ['b', 'c'];
['a', ...arr, 'd']
// ['a', 'b', 'c', 'd']
```

##### yield*

（这里是`Generator`章节的内容。推荐先看下一篇博客）

`yield*`后面跟的是一个可遍历的结构，它会调用该结构的遍历器接口。

```javascript
let generator = function* () {
  yield 1;
  yield* [2,3,4];
  yield 5;
};

var iterator = generator();

iterator.next() // { value: 1, done: false }
iterator.next() // { value: 2, done: false }
iterator.next() // { value: 3, done: false }
iterator.next() // { value: 4, done: false }
iterator.next() // { value: 5, done: false }
iterator.next() // { value: undefined, done: true }
```

##### 其他场合

由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口。下面是一些例子。

- `for...of`
- `Array.from()`
- `Map()`, `Set()`, `WeakMap()`, `WeakSet()`（比如`new Map([['a',1],['b',2]])`）
- `Promise.all()`
- `Promise.race()`

### 

### 兼容性

目前，所有 Firefox 的 Release 版本都已经支持 `for-of` 语句。Chrome 默认禁用了该语句，你可以在地址栏输入 `chrome://flags` 进入设置页面，然后勾选其中的 “Experimental JavaScript” 选项。微软的 Spartan 浏览器也支持该语句，但是 IE 不支持。如果你想在 Web 开发中使用该语句，而且需要兼容 IE 和 Safari 浏览器，你可以使用 [Babel](http://babeljs.io/) 或 Google 的 [Traceur](https://github.com/google/traceur-compiler#what-is-traceur) 这类编译器，来将 ES6 代码转换为 Web 友好的 ES5 代码。

对于服务器端，我们不需要任何编译器 – 可以在 io.js 中直接使用该语句，或者在 NodeJS 启动时使用 `--harmony` 启动选项。

### {done: true}

到此，今天的话题已经结束，但对于 `for-of` 的话题还没有结束。

在 ES6 中还有一个新对象，该对象可以与 `for-of` 语句完美地结合使用，今天我并没有提及该对象，因为这是下篇文章我们讨论的主题，我认为这个新对象是 ES6 中最大的特性。如果你还没有在 Python 或 C# 中接触过该对象，你会认为这太奇妙了，但这是编写一个迭代器的最简单的方法，而且它对代码重构非常有用，它还可能改变我们处理异步代码的方式。所以，接着关注我的下篇关于 Generator 的讨论。

### 下期预告：Generator 函数

`Symbol.iterator`方法的最简单实现，还是使用下一章要介绍的`Generator`函数。

```javascript
let myIterable = {
  [Symbol.iterator]: function* () {
    yield 1;
    yield 2;
    yield 3;
  }
}
[...myIterable] // [1, 2, 3]

// 或者采用下面的简洁写法

let obj = {
  * [Symbol.iterator]() {
    yield 'hello';
    yield 'world';
  }
};

for (let x of obj) {
  console.log(x);
}
// "hello"
// "world"
```

上面代码中，`Symbol.iterator`方法几乎不用部署任何代码，只要用 yield 命令给出每一步的返回值即可。





参考文章地址：<http://bubkoo.com/2015/06/15/es6-in-depth-iterators-and-the-for-of-loop/>