---
title: JavaScript中的new操作符
date: 2019-03-25 21:00:21
categories: JavaScript —— 原理篇
tags: 
- new操作符
- 实现原理
---

### 前言

玩过原型链继承（类继承）的娃应该都用过`var instance = new Class()`之类的代码。但是你真的了解`new`吗？

大家都知道`new`操作符用作实例化，但是实例化的过程中除去返回对象它还做了什么？



### 绑定this到实例

```javascript
function Student(name){
    console.log('赋值前-this', this);
    this.name = name;
    console.log('赋值后-this', this);
}
var student = new Student('Ming');
// 赋值前-this  Student{}
// 赋值后-this  Student{name: "Ming"}
```

由此可见，`new`操作符将构造函数`Student`中的`this`指向了`new Student()`生成的对象`student`。

<!--more-->

### 原型链连接

```javascript
function Test(name) {
  this.name = name
}
Test.prototype.sayName = function () {
    console.log(this.name)
}
const t = new Test('yck')
console.log(t.name) // 'yck'
t.sayName() // 'yck'
```

通过`new`操作符将实例`t`与构造函数`Test`连接起来。

通过`new`操作符将实例和构造函数连接起来。相当于`student.__proto__ = Student.prototype`。



当然我们可以通过`setPrototypeOf`完成。有兴趣的boy可以了解一下。

### 返回值

#### 原始值

```javascript
function Test(name) {
  this.name = name
  return 1
}
const t = new Test('yck')
console.log(t.name) // 'yck'
```

- **构造函数如果返回原始值（虽然例子中只有返回了 1，但是你可以试试其他的原始值，结果还是一样的），那么这个返回值毫无意义**

#### 对象

```javascript
function Test(name) {
  this.name = name
  console.log(this) // Test { name: 'yck' }
  return { age: 26 }
}
const t = new Test('yck')
console.log(t) // { age: 26 }
console.log(t.name) // 'undefined'
```

- **构造函数如果返回值为对象，那么这个返回值会被正常使用**

### 实现

我们在`coding`前先理一下思路

- 创建新对象
- 将构造函数的作用域赋给新对象（`obj.__proto__ = Base.prototype`）
- 执行构造函数中的代码（`Base.call(obj, ...args)`）
- 返回新对象

`OK, let's coding`

```javascript
/**
 * 模拟实现 new 操作符
 */
function create(Con, ...args) {
  let obj = {}
  Object.setPrototypeOf(obj, Con.prototype)
  let result = Con.apply(obj, args)
  return result instanceof Object ? result : obj
}
```

![](https://pic.superbed.cn/item/5c989eb83a213b041714ed22)



咱们来试一下这DIY的东西好不好用

```javascript
function Constructor(name){
	this.name = name;
}
constructor.prototype.fun = function(){
	console.log(this.name)
}
let a = create(Constructor, 'jack')
```

![](https://pic.superbed.cn/item/5c95e71f3a213b0417ef5ccf)

有没有发现，和咱们之前的一模一样欸。所以这东西就到此结束。

### 总结

最后总结一下

- 将`this`绑定到实例
- 原型链连接
- 返回值为对象