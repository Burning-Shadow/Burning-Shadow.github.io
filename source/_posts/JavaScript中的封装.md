---
title: JavaScript中的封装
categories: JavaScript设计模式
tags: 
 - 设计模式
 - 封装
---

在面向对象语言中会有各式各样的关键字，如`public`、`pcivate`、`protected`、`static`等等，设置了属性和方法的权限。

作为一门弱类型语言`JavaScript`显然不具备这样的确定权限关键字，所以我们采取了另外的方式来使得变量和方法的权限得以被区别开来

<!--more-->

### 提要

#### `private`

我们在`JavaScript`基础篇里讲到了闭包，其最大的特点之一就是可以访问外层作用域的变量以及函数。所以我们可以利用这一特性将属性封装为内部属性

> **`private`**：声明在函数内部的属性及方法在外界是无法访问到的

#### `public`

在原型链部分的学习中我们得知在原型上定义的方法可以被其子类所调用。

> **`public`**：在原型（`prototype`）上定义的属性/方法是公有属性/方法

#### `static`

在类的外部，通过点语法添加的属性、方法不会被初始化的`new`执行，故新创建的对象无法获取他们，只能通过类调用

> **`static`**：模式如`Book.isChinese = false`的类型（类名默认大写）

#### 特权方法

在函数内部通过`this`定义的属性&方法，通过其`new`出来的对象自身都具有一份。通过其实例就可以访问得到。同时还能访问到类（创建时）或对象自身的私有属性/方法

> **特权方法**：`this.getId = function(){console.log('this id my id')} `

### 栗子

我们可以举一个例子。创造一个名为“`Book`”的类且拥有`id`、`name`、`price`三个自定义属性

```javascript
var Book = function(id, name, price){
    var num = 1;	// 私有属性 private
    function checkId(){console.log('checkId')};		// 私有方法 private
    
    // 特权方法
    this.getName = function(){console.log()}
    this.getPrice = function(){console.log()}
    this.setName = function(){console.log()}
    this.setPrice = function(){console.log()}
    
    // 对象公有方法
    this.copy = function('We all have this method')
    
    // 对象公有属性
    this.id = id;
    
    // 构造器
    this.setName(name)
    this.setPrice(price)
}

// 静态类公有属性（对象无法访问）
Book.isChinese = true

// 静态类共有方法（对象不能访问）
Book.resetTime = function(){
    console.log('new Time')
}

Book.prototype = {
    // 公有属性
    isJSBook :false
    
    //公有方法
    display: function(){ console.log('Public method') }
}
```

我们可以写代码测试一下

```javascript
var instance = new Book(11, 'JavaScript', 50)
console.log(b.num)			// undefined
console.log(b.isJSBook)		// false
console.log(b.id)			// 11
console.log(b.inChinese)	// undefined
console.log(Book.isChinese)	// true
Book.resetTime()			// new Time
```

今天要讲的就是这些啦，88。