---
title: JavaScript之深浅拷贝
date: 2019-04-06 12:24:03
categories: JavaScript
tags:
 - 深浅拷贝
---

### 前言

首先咱们先理一下这玩意儿的概念

> 浅拷贝：将对象的各个属性进行依次复制，并不会进行递归复制，也就是说**只会赋值目标对象的第一层属性**。
>
> 深拷贝：**递归拷贝目标对象的所有属性**。

咱们可以这么理解：浅拷贝就像人口普查，只往上看这一代。深拷贝则是入党审核，你家庭案底都得查得清清楚楚。

<!--more-->

那么咱们正式开始

### 应用场景

当我们需要大量复制某些对象而其却可以完全独立，互不干扰的时候我们就需要用到拷贝。

而深拷贝还是浅拷贝则需要根据我们实际需要而定。如果涉及到继承之类的东西我们当然需要深层拷贝。而如果只是属性方面的复制还是推荐使用浅拷贝。

### 浅拷贝

#### JQ中的$.extend({}, obj)

这个就不再多说咯，调用官方API就可以轻松完成。不过如果小伙伴们使用框架，比如自带DOM渲染的Vue，由于他是MVVM框架，尽量避免直接操作DOM，所以大部分玩家不会选择直接引入jq。所以我们下面介绍一下其他的实现方式

#### for in暴力赋值

最简单的方法就是通过`for...in`循环遍历赋值。如此我们不会误将引用赋给另一个对象，自然也不用担心对象之间的独立性

```javascript
function shallowCopy( source ) {
    var target = {};
    for (var key in source) {
        if (source.hasOwnProperty(key)) {
            target[key] = source[key];
        }
    }
    return target;
}
```

 里边我们涉及到了一个函数：`hasOwnProperty()`。他可以**防止向上继续进行拷贝，只对本层处理**。

#### Object.assign(target, obj1, obj2, ....)

- `target`：目标对象
- `obj1`：源对象1
- `obj2`：源对象2

其作用是将源对象的属性复制到目标对象中去。

这东西主要用来将多个对象的属性合并到一个对象上去，但是咱们在这里也可以用走浅拷贝。

```javascript
let obj = { name: '程序猿', age:{child: 12} }
let copy = Object.assign({}, obj);
copy.name = '单身狗'
copy.age.child = 24
console.log(obj) // { name: '程序猿', age:{child: 24} }  当用来修改的属性包含对象时就会统一改变。
```

这个例子就可以形象地说明这东西的特点啦：

有没有注意到，**我们第一层的属性（`name`、`age`）独立，而第二层的属性（`child`）则未能完成上面提到的“互不干扰”。**这不正好符合我们的思路吗

如果需要深层拷贝的话，咱加个递归不就好了吗~(`4.2`)

#### 展开运算符（...）

```javascript
let a = {
    age: 1
}
let b = {...a}
a.age = 2
console.log(b.age) // 1
```



### 深拷贝

#### for in循环递归

```javascript
function deepCopy( source ) {
    let target = Array.isArray( source ) ? [] : {}
    for ( var k in source ) {
        if ( typeof source[ k ] === 'object' ) {
            target[ k ] = deepCopy( source[ k ] )
        } else {
            target[ k ] = source[ k ]
        }
    }
    return target
}
```

#### Object.assign循环递归

```javascript
function deepCopy( source ) {
    var target = Object.assign({}, source)
    for(var key in source){
        target[key] = deepCopy(source[key])
    }
    return target
}
```



#### 开挂法

```javascript
let copy = JSON.parse(JSON.stringify(obj));
```

将对象转化为`json`字符串再转换为`json`对象。**但是有两个弊端：**

- 若你的对象里有函数，那你这函数没法被拷贝下来
- 无法拷贝`obj`原型链上的属性和方法。

### 刨根问底

#### 深浅拷贝——堆和栈的区别

身前拷贝的主要区别还是内存中的存储类型不同。

> **基本类型**（`undefined`、`boolean`、`number`、`string`、`null`）**存放在栈中**
>
> **引用类型**（`object`、`Array`、`function`）则**存放在堆中**

为什么说这个呢？咱们细细道来

##### 基本数据类型不可变

**所有的基本数据类型的值都不可更改的**：任何方法都无法改变一个原始值。

可能有的Boy会问：我这么操作不也就改了吗

```javascript
var num = 12;
num = 13;
console.log(num)		// 13
```

可是事实上这是对基本类型的重新赋值，并不算更改。



**基于以上，引用类型的变量实际上是一个存放在栈内存的指针，指向堆内存中的地址。**



> 另外，基本类型的值比较是值的比较
>
> 引用类型的值比较是引用地址的比较（即使其中的属性、各索引元素完全相等亦不相等。只有引用基于同一个基对象时才算相等）

