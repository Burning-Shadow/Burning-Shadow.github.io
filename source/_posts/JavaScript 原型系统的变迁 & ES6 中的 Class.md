---
title: JavaScript 原型系统的变迁 & ES6 中的 Class
date: 2019-03-26 01:23:10
categories: JavaScript —— 原理篇
tags:
 - Class语法
 - 继承
 - ES6
 - 原型
---

这篇文章是我从`Segmentfault`上转载而来，颠覆了我对之前原型、继承、公私有的理解，转载此文，并在此表示对作者深深的敬意。														—— 2019.3.26       Siir

<!--more-->

### 开始前对 Class 的一点强调

ES6中的`class`可以看作只是一个语法糖，绝大部分功能都可以用ES5实现，并且，**类和模块的内部，默认就是严格模式，所以不需要使用use strict指定运行模式**。



```javascript
// ES5
function P (x,y){
    this.x = x;
    this.y = y;
}
P.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
};
var a = new P(1, 2);

// ES6
class P {
    constructor(x, y){
        this.x = x;
        this.y = y;
    }
    toString(){
        return '(' + this.x + ', ' + this.y + ')';
    }
}
let a = new P(1, 2);
```

ES6中类的所有方法都是定义在`prototype`属性上。调用类实例的方法，其实就是调用原型上的方法

```javascript
class P {
    constructor(){ ... }
    toString(){ ... }
    toNumber(){ ... }
}
// 等同于
P.prototyoe = {
    constructor(){ ... },
    toString(){ ... },
    toNumber(){ ... }
}

let a = new P();
a.constructor === P.prototype.constructor; // true
```

### 概述

JavaScript 的原型系统是最初就有的语言设计。但随着 ES 标准的进化和新特性的添加。它也一直在不停进化。这篇文章的目的就是梳理一下早期到 ES5 和现在 ES6，新特性的加入对原型系统的影响。

如果你对原型的理解还停留在 `function + new` 这个层面而不知道更深入的操作原型链的技巧，或者你想了解 ES6 class 的知识，相信本文会有所帮助。

这篇文章是我学习 You Don't Know JS 的副产品，推荐任何想系统性地学习 JavaScript 的人去阅读此书。

### JavaScript 原型简述

很多人应该都对原型（prototype）不陌生。简单地说，JavaScript 是基于原型的语言。当我们调用一个对象的属性时，如果对象没有该属性，JavaScript 解释器就会从对象的原型对象上去找该属性，如果原型上也没有该属性，那就去找原型的原型。这种属性查找的方式被称为原型链（prototype chain）。

对象的原型是没有公开的属性名去访问的（下文再谈 `__proto__` 属性）。以下为了方便称呼，我把一个对象内部对原型的引用称为 [[Prototype]]。

JavaScript 没有类的概念，原型链的设定就是少数能够让多个对象共享属性和方法，甚至模拟继承的方式。在 ES5 以前，如果我们想设置对象的 [[Prototype]]，只能通过 `new` 关键字，比如：

```javascript
function User() {
  this._name = 'David'
}

User.prototype.getName = function() {
  return this._name
}

var user = new User()
user.getName()                  // "David"
user.hasOwnProperty('getName')  // false
```

当 `User` 函数被 `new` 关键字调用时，它就类似于一个构造函数，其生成的对象的 [[Prototype]] 会引用 `User.prototype` 。因为 `User.prototype` 也是一个对象，它的 [[Prototype]] 是 `Object.prototype` 。

一般我们对这种构造函数命名都会采用 CamelCase ，并把它称呼为“类”，这不仅是为了跟 OOP 的理念保持一致，也是因为 JavaScript 的内建“类”也是这种命名。

由 `SomeClass` 生成的对象，其 [[Prototype]] 是 `SomeClass.prototype`。除了稍显繁琐，这套逻辑是可以自圆其说的，比如：

1. 我们用 `{..}` 创建的对象的 [[Prototype]] 都是 `Object.prototype`，也是原型链的顶点。
2. 数组的 [[Prototype]] 是 `Array.prototype` 。
3. 字符串的 [[Prototype]] 是 `String.prototype` 。
4. `Array.prototype` 和 `String.prototype` 的 [[Prototype]] 是 `Object.prototype` 。

### 模拟继承

模拟继承是自定义原型链的典型使用场景。但如果用 `new` 的方式则比较麻烦。一种常见的解法是：子类的 `prototype` 等于父类的实例。这就涉及到定义子类的时候调用父类的构造函数。为了避免父类的构造函数在类定义过程中的潜在影响，我们一般会建造一个临时类去做代替父类 `new` 的过程。

```javascript
function Parent() {}
function Child() {}

function createSubProto(proto) {
  // fn 在这里就是临时类
  var fn = function() {}
  fn.prototype = proto
  return new fn()
}

Child.prototype = createSubProto(Parent.prototype)
Child.prototype.constructor = Child

var child = new Child()
child instanceof Child   // true
child instanceof Parent  // true
```

### ES5: 自由地操控原型链

既然原型链本质上只是建立对象之间的关联，那我们可不可以直接操作对象的 [[Prototype]] 呢？

在 ES5（准确的说是 5.1）之前，我们没有办法直接获取对象的原型，只能通过 [[Prototype]] 的 `constructor`。

```javascript
var user = new User()
user.constructor.prototype          // User
user.hasOwnProperty('constructor')  // false
```

类可以通过 `prototype` 属性获取生成的对象的 [[Prototype]]。[[Prototype]] 里的 `constructor` 属性又会反过来引用函数本身。因为 `user` 的原型是 `User.prototype` ，它自然也能够通过 `constructor` 获取到 `User` 函数，进而获取到自己的 [[Prototype]]。比较绕是吧？

ES5.1 之后加了几个新的 API 帮助我们操作对象的 [[Prototype]]，自此以后 JavaScript 才真的有自由操控原型的能力。它们是：

- `Object.prototype.isPrototypeOf`
- `Object.create`
- `Object.getPrototypeOf`
- `Object.setPrototypeOf`

注：以上方法并不完全是 ES5.1 的，`isPrototypeOf` 是 ES3 就有的，`setPrototypeOf` 是 ES6 才有的。但它们的规范都在 ES6 中修改了一部分。

下面的例子里，`Object.create` 创建 `child` 对象，并把 [[Prototype]] 设置为 `parent` 对象。`Object.getPrototypeOf` 可以直接获取对象的 [[Prototype]]。`isPrototypeOf` 能够判断一个对象是否在另一个对象的原型链上。

```javascript
var parent = {
  _name: 'David',
  getName: function() { return this._name },
}

var child = Object.create(parent)

Object.getPrototypeOf(child)           // parent
parent.isPrototypeOf(child)            // true
Object.prototype.isPrototypeOf(child)  // true
child instanceof Object                // true
```

既然有 `Object.getPrototypeOf`，自然也有 `Object.setPrototypeOf` 。这个函数可以修改任何对象的 [[Prototype]] ，包括内建类型。

```javascript
var anotherParent = {
  name: 'Alex'
}

Object.setPrototypeOf(child, anotherParent)
Object.getPrototypeOf(child)  // anotherParent

// 修改数组的 [[Prototype]]
var a = []
Object.setPrototypeOf(a, anotherParent)
a instanceof Array        // false
Object.getPrototypeOf(a)  // anotherParent
```

灵活使用以上的几个方法，我们可以非常轻松地创建原型链，或者在已知原型链中插入自定义的对象，玩法只取决于想象力。我们以此修改一下上面的模拟继承的例子：

```javascript
function Parent() {}
function Child() {}

Child.prototype = Object.create(Parent.prototype)
Child.prototype.constructor = Child
```

因为 `Object.create(..)` 传入的参数会作为 [[Prototype]] ，所以这里有一个有意思的小技巧。我们可以用 `Object.create(null)` 创建一个没有任何属性的对象。这个技巧适合做 proxy 对象，有点类似 Ruby 中的 `BasicObject`。

### 尴尬的私生子 `__proto__`

说到操作 [[Prototype]] 就不得不提 `__proto__` 。这个属性是一个 getter/setter ，可以用来获取和设置任意对象的 [[Prototype]] 。

```javascript
child.__proto__           // equal to Object.getPrototypeOf(child)
child.__proto__ = parent  // equal to Object.setPrototypeOf(child, parent)
```

它本来不是 ES 的标准，无奈众多浏览器早早地都实现了这个属性，而且应用得还挺广泛的。到了 ES6 为了向下兼容性只好接纳它成为标准的一部分。这是典型的现实倒逼标准的例子。

看看 MDN 的描述都充满了怨念。

> The use of **proto** is controversial, and has been discouraged. It was never originally included in the EcmaScript language spec, but modern browsers decided to implement it anyway. Only recently, the **proto**property has been standardized in the ECMAScript 6 language specification for web browsers to ensure compatibility, so will be supported into the future. It is deprecated in favor of Object.getPrototypeOf/Reflect.getPrototypeOf and Object.setPrototypeOf/Reflect.setPrototypeOf (though still, setting the [[Prototype]] of an object is a slow operation that should be avoided if performance is a concern).

`__proto__` 是不被推荐的用法。大部分情况下我们仍然应该用 `Object.getPrototypeOf` 和 `Object.setPrototypeOf` 。什么是少数情况，待会再讲。

### Class 语法糖

```javascript
class User {
  constructor(firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName
  }

  fullName() {
    return `${this.firstName} ${this.lastName}`
  }
}

let user = new User('David', 'Chen')
user.fullName()  // David Chen
```

以上代码的意思和 ES5 语法是一个意思

```javascript
function User(firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}

User.prototype.fullName = function() {
  return '' + this.firstName + this.lastName
}
```

我个人理解来看就是相当于一个组合式的继承。将属性写在构造函数之中，而将方法放到原型链上，以达到独立属性，共享方法的优势。



ES6 并未改变 JavaScript 基于原型的本质，只是在此基础上提供了一些语法糖。比如`class`、`extends`、`super`、`static`。他们大都可以转换为等价的 ES5 语法

我们来看另一个例子

```javascript
class Child extends Parent {
  constructor(firstName, lastName, age) {
    super(firstName, lastName)
    this.age = age
  }
}
```

此基本类等价于

```javascript
function Child(firstName, lastName, age) {
  Parent.call(this, firstName, lastName)
  this.age = age
}

Child.prototype = Object.create(Parent.prototype)
Child.constructor = Child
```

### Extends

因为语言内部设计原因，我们没有办法自定义一个类来继承 JavaScript 的内建类的。继承类往往会有各种问题。ES6 的 `extends` 的最大的卖点，就是不仅可以继承自定义类，**还可以继承 JavaScript 的内建类**。

```javascript
class MyArray extends Array {
}
```

这种方式可以让开发者继承内建类的功能创造出符合自己想要的类。所有 Array 已有的属性和方法都会对继承类生效。这确实是个不错的诱惑，也是继承最大的吸引力。



但现实总是悲催的。`extends` 内建类会引发一些奇怪的问题，很多属性和方法没办法在继承类中正常工作。

```javascript
var a = new Array(1, 2, 3)
a.length  // 3

var b = new MyArray(1, 2, 3)
b.length  // 0
```

如果说语法糖可以用 Babel.js 这种 transpiler 去编译成 ES5 解决 ，扩充的 API 可以用 polyfill 解决，但是这种内建类的继承机制显然是需要浏览器支持的。而目前唯一支持这个特性的浏览器是………… Microsoft Edge 。

好在这并不是什么致命的问题。大多数此类需求都可以用封装类去解决，无非是多写一点 wrapper API 而已。而且个人认为封装和组合反而是比继承更灵活的解决方案。

### super 带来的新概念（坑？）

#### super 在 constructor 和普通方法里的不同

在 constructor 里面，`super` 的用法是 `super(..)`。它相当于一个函数，调用它等于调用父类的 constructor 。但在普通方法里面，`super` 的用法是 `super.prop` 或者 `super.method()`。它相当于一个指向对象的 [[Prototype]] 的属性。这是 ES6 标准的规定。

```javascript
class Parent {
  constructor(firstName, lastName) {
    this.firstName = firstName
    this.lastName = lastName
  }

  fullName() {
    return `${this.firstName} ${this.lastName}`
  }
}

class Child extends Parent {
  constructor(firstName, lastName, age) {
    super(firstName, lastName)
    this.age = age
  }

  fullName() {
    return `${super.fullName()} (${this.age})`
  }
}
```

注意：Babel.js 对方法里调用 `super(..)` 也能编译出正确的结果，但这应该是 Babel.js 的 bug ，我们不该以此得出 `super(..)` 也可以在非 constructor 里用的结论。

#### super 在子类的 constructor 里必须先于 this 调用

如果写子类的 `constructor` 需要操作 `this` ，那么 `super` 必须先调用！这是 ES6 的规则。所以写子类的 `constructor` 时尽量把 `super` 写在第一行。

```javascript
class Child extends Parent {
  constructor() {
    this.xxx()  // invalid
    super()
  }
}
```

#### super 是编译时确定，不是运行时确定

什么意思呢？先看代码：

```javascript
class Child extends Parent {
  fullName() {
    super.fullName()
  }
}
```

以上代码中 `fullName` 方法的 ES5 等价代码是

```javascript
fullName() {
  Parent.prototype.fullName.call(this)
}
```

而不是

```javascript
fullName() {
  Object.getPrototypeOf(this).fullName.call(this)
}
```

这就是 `super` 编译时确定的特性。不过为什么要这样设计？个人理解是，函数的 `this` 只有在运行时才能确定。因此在运行时根据 `this` 的原型链去获得上层方法并不太符合 class 的常规思维，在某些情况下更容易产生错误。比如 `child.fullName.call(anotherObj)` 。

#### super 对 static 的影响，和类的原型链

`static` 相当于类方法。因为编译时确定的特性，以下代码中：

```javascript
class Child extends Parent {
  static findAll() {
    return super.findAll()
  }
}
```

`findAll` 的 ES5 等价代码是：

```javascript
findAll() {
  return Parent.findAll()
}
```

这就是 `super` 编译时确定的特性。不过为什么要这样设计？个人理解是，函数的 `this` 只有在运行时才能确定。因此在运行时根据 `this` 的原型链去获得上层方法并不太符合 class 的常规思维，在某些情况下更容易产生错误。比如 `child.fullName.call(anotherObj)` 。

#### super 对 static 的影响，和类的原型链

`static` 相当于类方法。因为编译时确定的特性，以下代码中：

```javascript
class Child extends Parent {
  static findAll() {
    return super.findAll()
  }
}
```

`findAll` 的 ES5 等价代码是：

```javascript
findAll() {
  return Parent.findAll()
}
```

`static` 貌似和原型链没关系，但这不妨碍我们讨论一个问题：类的原型链是怎样的？我没查到相关的资料，不过我们可以测试一下：

```javascript
Object.getPrototypeOf(Child) === Parent             // true
Object.getPrototypeOf(Parent) === Object            // false
Object.getPrototypeOf(Parent) === Object.prototype  // false

proto = Object.getPrototypeOf(Parent)
typeof proto                             // function
proto.toString()                         // function () {}
proto === Object.getPrototypeOf(Object)  // true
proto === Object.getPrototypeOf(String)  // true

new proto()  //TypeError: function () {} is not a constructor
```

可见自定义类的话，子类的 [[Prototype]] 是父类，而所有顶层类的 [[Prototype]] 都是同一个函数对象，不管是内建类如 `Object` 还是自定义类如 `Parent` 。但这个函数是不能用 `new` 关键字初始化的。虽然这种设计没有 Ruby 的对象模型那么巧妙，不过也是能够自圆其说的。

#### 直接定义 object 并设定 [[Prototype]]

除了通过 `class` 和 `extends` 的语法设定 [[Prototype]] 之外，现在定义对象也可以直接设定 [[Prototype]] 了。这就要用到 `__proto__` 属性了。“定义对象并设置 [[Prototype]]” 是唯一建议用 `__proto__` 的地方。另外，另外注意 `super` 只有在 `method() {}` 这种语法下才能用。

```javascript
let parent = {
  method1() { .. },
  method2() { .. },
}

let child = {
  __proto__: parent,

  // valid
  method1() {
    return super.method1()
  },

  // invalid
  method2: function() {
    return super.method2()
  },
}
```

### 总结

avaScript 的原型是很有意思的设计，从某种程度上说它是更加纯粹的面向对象设计（而不是面向类的设计）。ES5 和 ES6 加入的 API 能更有效地操控原型链。语言层面支持的 `class` 也能让忠于类设计的开发者用更加统一的方式去设计类。虽然目前 `class` 仅仅提供了一些基本功能。但随着标准的进步，相信它还会扩充出更多的功能。

本文的主题是原型系统的变迁，所以并没有涉及 getter/setter 和 `defineProperty` 对原型链的影响。想系统地学习原型，你可以去看 [You Don't Know JS: this & Object Prototypes](https://github.com/getify/You-Dont-Know-JS/tree/master/this%20&%20object%20prototypes) 。





原文地址：<https://segmentfault.com/a/1190000003798438>



