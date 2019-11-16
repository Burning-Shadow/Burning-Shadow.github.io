---
title: 浅谈 instanceof 和 typeof 原理
date: 2019-03-06 10:13:00
categories: JavaScript —— 原理篇
tags: 
- typeof
- instanceof
---

对于类型判断我们很多人都知道`null`和`instanceof`这两个方法。但是同时这两个原生自带的方法也有自己的局限性。

`typeof`对于基本类型，除了`null`都可以正确显示

>typeof 1 // 'number'
>typeof '1' // 'string'
>typeof undefined // 'undefined'
>typeof true // 'boolean'
>typeof Symbol() // 'symbol'
>typeof b // b 没有声明，但是还会显示 undefined

而对于对象，除了函数都会显示`object`

>typeof [] // 'object'
>typeof {} // 'object'
>typeof console.log // 'function'

而`instanceof`则大多用在继承那一章节中对实例是否从属于某个类进行判断。

下面我们来谈一谈他们的机制8~

<!--more-->

## typeof 原理

`typeof` 一般被用于判断一个变量的类型，我们可以利用 `typeof` 来判断`number`,  `string`,  `object`,` boolean`,  `function`, `undefined`,  `symbol` 这七种类型，这种判断能帮助我们搞定一些问题，比如在判断不是 `object` 类型的数据的时候，`typeof`能比较清楚的告诉我们具体是哪一类的类型。但是，很遗憾的一点是，**`typeof `在判断一个 `object`的数据的时候只能告诉我们这个数据是 `object`, 而不能细致的具体到是哪一种 `object`**

```javascript
let s = new String('abc');
typeof s === 'object'// true
s instanceof String // true
```
要想判断一个数据具体是哪一种 `object` 的时候，我们需要利用 `instanceof` 这个操作符来判断，这个我们后面会说到。
来谈谈关于 `typeof` 的原理吧，我们可以先想一个很有意思的问题，`js` 在底层是怎么存储数据的类型信息呢？或者说，一个 `js` 的变量，在它的底层实现中，它的类型信息是怎么实现的呢？

<!--more-->

其实，**js 在底层存储变量的时候，会在变量的机器码的低位1-3位存储其类型信息**

- 000：对象
- 010：浮点数
- 100：字符串
- 110：布尔
- 1：整数

but, 对于 `undefined` 和 `null` 来说，这两个值的信息存储是有点特殊的。

`null`：所有机器码均为0
`undefined`：用 −2^30 整数来表示
所以，`typeof` 在判断 `null` 的时候就出现问题了，**由于`null`的所有机器码均为0，因此直接被当做了对象来看待**。
然而用 `instanceof` 来判断的话

```javascript
null instanceof null // TypeError: Right-hand side of 'instanceof' is not an object
```

`null` 直接被判断为不是`object`，这也是` JavaScript`的历史遗留`bug`，可以参考`typeof`。



### 番外篇——神奇的null（有坑，未填）

![神奇的null](https://pic.superbed.cn/item/5c94f5683a213b0417e591d4)

- 当我们在控制台输入`typeof null`时会打印`object`。这也是一个存在了很久的`bug`
  - PS：为什么会出现这种情况呢？因为在 JS 的最初版本中，使用的是 32 位系统，为了性能考虑使用低位存储了变量的类型信息，`000` 开头代表是对象，然而 `null` 表示为全零，所以将它错误的判断为 `object` 。虽然现在的内部类型判断代码已经改变了，但是对于这个 Bug 却是一直流传下来。
- 当我们**在控制台输入 `null == undefined` 时会打印 `true`** 。因为**`undefined` 派生自 `null`**。所以当换成全等符号后值就变成了`false`。
  - 有意思的一点是，将二者强转为`Number`型时`null`值为`0`而`undefined`值为`NaN`。
- 而作为基本类型之一的`null`，当用户打印`null instanceof null`时却报了错（`TypeError: Right-hand side of 'instanceof' is not an object`），这也是`JavaScript`历史遗留的`bug`，为了防止修复造成的更多`bug`所以一直没有修复...
- 而当我们在控制台输入 `null instanceof Object` 时则会打印 `false` ，这和咱们输入 `typeof null` 的值 `"object"`大相径庭，具体原理我也没摸清，以后再填坑吧。
  - 【填坑】后来我听到了一种比较合理的解释，`null`我们可以理解为一个占位符，并不由`Object`自原型链继承而来，故`null instanceof Object`结果为`false`。另一方面，由于它是一个对象的占位符，故判断类型时将其自动归属于`Object`大类中。







因此在用 `typeof` 来判断变量类型的时候，我们需要注意，**最好是用 `typeof` 来判断基本数据类型（包括`symbol`），避免对 `null` 的判断**。

还有一个不错的判断类型的方法，就是**`Object.prototype.toString`**，我们可以利用这个方法来对一个变量的类型来进行比较准确的判断

```javascript
Object.prototype.toString.call(1) // "[object Number]"

Object.prototype.toString.call('hi') // "[object String]"

Object.prototype.toString.call({a:'hi'}) // "[object Object]"

Object.prototype.toString.call([1,'a']) // "[object Array]"

Object.prototype.toString.call(true) // "[object Boolean]"

Object.prototype.toString.call(() => {}) // "[object Function]"

Object.prototype.toString.call(null) // "[object Null]"

Object.prototype.toString.call(undefined) // "[object Undefined]"

Object.prototype.toString.call(Symbol(1)) // "[object Symbol]"


```
## instanceof 原理
之前我们提到了`instanceof`来判断对象的具体类型，其实`instanceof`主要的作用就是判断一个实例是否属于某种类型

```javascript
let person = function () {
}
let nicole = new person()
nicole instanceof person // true
```
当然，`instanceof`也可以判断一个实例是否是其父类型或者祖先类型的实例。

```javascript
let person = function () {
}
let programmer = function () {
}
programmer.prototype = new person()
let nicole = new programmer()
nicole instanceof person // true
nicole instanceof programmer // true

```
这是`instanceof`的用法，但是`instanceof`的原理是什么呢？根据`ECMAScript`语言规范，我梳理了一下大概的思路，然后整理了一段代码如下

```javascript
function new_instance_of(leftVaule, rightVaule) { 
    let rightProto = rightVaule.prototype; // 取右表达式的 prototype 值
    leftVaule = leftVaule.__proto__; // 取左表达式的__proto__值
    while (true) {
    	if (leftVaule === null) {
            return false;	
        }
        if (leftVaule === rightProto) {
            return true;	
        } 
        leftVaule = leftVaule.__proto__ 
    }
}
```
其实 **`instanceof`主要的实现原理就是只要右边变量的 prototype 在左边变量的原型链上即可。因此，`instanceof`在查找的过程中会遍历左边变量的原型链，直到找到右边变量的`prototype`，如果查找失败，则会返回 `false`**，告诉我们左边变量并非是右边变量的实例。
看几个很有趣的例子

```javascript
function Foo() {
}

Object instanceof Object // true
Function instanceof Function // true
Function instanceof Object // true
Foo instanceof Foo // false
Foo instanceof Object // true
Foo instanceof Function // true

```
要想全部理解`instanceof`的原理，除了我们刚刚提到的实现原理，我们还需要知道 `JavaScript`的原型继承原理。

关于原型继承的原理，我简单用一张图来表示

![一张图搞懂原型链](https://pic.superbed.cn/item/5c93bda83a213b0417da42a9)

我们知道每个 JavaScript 对象均有一个隐式的`__proto__ `原型属性，而显式的原型属性是`prototype`，只有 `Object.prototype.__proto__` 属性在未修改的情况下为`null`值。根据图上的原理，我们来梳理上面提到的几个有趣的`instanceof`使用的例子。
- `Object instanceof Object`
    - 由图可知，`Object`的`prototype`属性是`Object.prototype`, 而由于`Object`本身是一个函数，由 `Function`所创建，所以`Object.__proto__`的值是`Function.prototype`，而`Function.prototype`的`__proto__`属性是`Object.prototype`，所以我们可以判断出，`Object instanceof Object`的结果是`true`。用代码简单的表示一下

```javascript
leftValue = Object.__proto__ = Function.prototype;
rightValue = Object.prototype;
// 第一次判断
leftValue != rightValue
leftValue = Function.prototype.__proto__ = Object.prototype
// 第二次判断
leftValue === rightValue
// 返回 true
```
`Function instanceof Function`和`Function instanceof Object`的运行过程与`Object instanceof Object `类似，故不再详说。

- `Foo instanceof Foo`
    - `Foo`函数的`prototype`属性是`Foo.prototype`，而 `Foo`的`__proto__` 属性是`Function.prototype`，由图可知，`Foo`的原型链上并没有`Foo.prototype`，因此`Foo instanceof Foo`也就返回` false` 。

```javascript
leftValue = Foo, rightValue = Foo
leftValue = Foo.__proto = Function.prototype
rightValue = Foo.prototype
// 第一次判断
leftValue != rightValue
leftValue = Function.prototype.__proto__ = Object.prototype
// 第二次判断
leftValue != rightValue
leftValue = Object.prototype = null
// 第三次判断
leftValue === null
// 返回 false
```
- `Foo instanceof Object`

```javascript
leftValue = Foo, rightValue = Object
leftValue = Foo.__proto__ = Function.prototype
rightValue = Object.prototype
// 第一次判断
leftValue != rightValue
leftValue = Function.prototype.__proto__ = Object.prototype
// 第二次判断
leftValue === rightValue
// 返回 true 
```
- `Foo instanceof Function`

```javascript
leftValue = Foo, rightValue = Function
leftValue = Foo.__proto__ = Function.prototype
rightValue = Function.prototype
// 第一次判断
leftValue === rightValue
// 返回 true 
```
## 总结
简单来说，我们使用`typeof`来判断基本数据类型是`ok`的，不过需要注意当用`typeof`来判断`null`类型时的问题，如果想要判断一个对象的具体类型可以考虑用`instanceof`，但是`instanceof`也可能判断不准确，比如一个数组，他可以被`instanceof`判断为`Object`所以我们要想比较准确的判断对象实例的类型时，可以采取 `Object.prototype.toString.call`方法。


