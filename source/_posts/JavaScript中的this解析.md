---
title: JavaScript中的this解析
date: 2019-03-24 21:00:10
categories: JavaScript
tags:
 - this指向
---

一张图放在前面

![this绑定](https://pic.superbed.cn/item/5c946ad83a213b0417dfd9ca)

<!--more-->

### 默认绑定

**独立函数调用**时遵循默认绑定的原则。一般情况下若无其他规则出现则默认将`this`绑定到全局对象上。
```javascript
function foo(){
	var a = 3;
	console.log(this.a)
}
var a = 2;
foo()	// 2
```
### 隐式绑定
若调用位置有上下文对象就遵循隐式绑定（如example.foo()）
```javascript
function foo(){
	console.log(this.a)
}
var obj = {a:2,  foo:foo}
var a = 'global'
obj.foo()		// 2
var bar = obj.foo
bar()		// 'global
```
如图。之前我们直接调用`foo`函数时因为其具有上下文对象，所以正确打印，`this`此时正指向此对象(`obj`)。
可是之后的`bar`为什么又无法正确打印出对应的结果呢？
首先我们看，题目中用一个变量`bar`存储了`obj.foo`。所以为完成此赋值操作引擎会对`obj.foo`进行`RHS`查找，并找出其对应的键`foo`。继而再对值`foo`进行`RHS`查找，得到`foo`函数。
可是此时对其进行调用时，由于无上下文对象（`bar`本身的调用并不是靠"`.`"出来的）所以就造成了隐式丢失。故采取默认绑定规则，绑定在全局的`a`上。
或者我们换一种说法：`bar`引用的时`foo`函数本身。因而此时的`bar`其实是一个不带任何修饰的函数调用，自然采取默认绑定规则。



Ps：隐式丢失后会默认绑定在`window`上

```javascript
var a = "global";
function afun(){
    function foo(){
        console.log(this.a)
    }
    var obj = {a:2,  foo:foo}
    var a = 'innerglobal'
    obj.foo()		// 2
    var bar = obj.foo
    bar()		// 'global
}
afun()
2
global
```

#### 这题怎么解决
上面我们讲到了，隐式绑定的规则说通俗一点就是必须用"`.`"操作符将其函数与相应对象绑定。所以我们改成这样就可以啦
```javascript
function foo(){
	console.log(this.a)
}
var obj = {a:2,  foo:foo}
var a = 'global'
obj.foo()		// 2
var bar = obj
bar.foo()		    // 2
```
此例中前半部分都一样。一直进行`RHS`查找，直到将`obj`的调用地址赋给`bar`。也就是说`bar`此时具有了一个`obj`的引用。在此后通过`.`操作符操作时就不会再发生上面所说的隐式丢失了.

##### 原理
>对象属性引用链只有上一层或者说最后一层再调用位置中起作用
##### 参数传递所引起的隐式丢失

```javascript
function foo(){
	console,log(a)
}
function dooFoo(fn){
	fn()	// 调用位置
}
var obj = {a:2,  foo:foo}
var a = 'global'
dooFoo(obj.foo)		// 'global'
```
我们分析一下。作为参数传递进`dooFoo`的`obj.foo`，引擎会对其进行`RHS`搜索。不信我们删掉`foo`看一下
![在这里插入图片描述](https://pic.superbed.cn/item/5c9459b83a213b0417df2980)



喏，这个`ReferenceError`的报错就能充分说明问题了。

> 不成功的`RHS`引用会导致抛出 `ReferenceError` 异常。
>
> 不成功的 `LHS` 引用会导致自动隐式地创建一个全局变量(非严格模式下)。该变量使用 `LHS` 引用的目标作为标识符，或者抛 出 `ReferenceError` 异常(严格模式下)。

既然是`RHS`那么我们会得到一个`foo`函数的引用。但是此引用是脱离了上下文的，自然会发生隐式丢失。



**`JavaScript`的内置库函数也一样，所以少用这种传入参数的方式，容易造成隐式丢失**

### 显式绑定
隐式绑定这种由JavaScript内部机制造成的宫斗剧一般的勾心斗角显然不适合我这种单纯的boy。而正好有另一种绑定方式，简单粗暴易懂，让人一眼看出`this`作用域。
#### apply、call
在用此方法之前我建议好兄弟们去看一下这俩函数的[相关内容](https://blog.csdn.net/qq_38722097/article/details/88126276)
```javascript
function foo(){
	console,log(a)
}
var obj = {a: 2}
foo.call(obj)	// 2
```
**但是显式绑定仍然无法解决丢失绑定问题**
#### 硬绑定
##### 创建一个可以重复使用的辅助函数
```javascript
function foo(something){
	console.log(this.a, something)
	return this.a + something
}
function bind(fn, obj){
	return function(){
		return fn.apply(obj, arguments)
	}
}
var obj = {a:2}
var b = bar(3)	// 2 3
console.log(b)	// 5
```
如图，我们每次在`bind`函数上将传入的函数硬性绑定在其对象上，如此一来无论如何调用`bar`，都会手动在`obj`上调用`fn`
##### Function.prototype.bind
```javascript
function foo(something){
	console.log(this.a, something)
	return this.a + something
}
var obj = {a:2}
var b = bar(3)	// 2 3
console.log(b)	// 5
```
`bind()`会返回一个硬编码的新函数，它会把你指定的参数设置为`this`的上下文并调用原始函数

#### API调用参数指定this
一些函数会提供一个可选参数作为你的“上下文”，以达到确保回调函数使用指定`this`的目的
```javascript
function foo(el){
	console.log(el, this.id)
}
var obj = {id:'awesome'}
[1, 2, 3].forEach(foo, obj)
```
### new绑定
#### 写在前面
>JavaScript中没有构造函数，只有对函数的构造调用
>发生函数的构造调用时，自动执行以下操作
- 创建一个全新的对象
- 此对象会被执行`[[prototype]]`链接
- 此新对象会绑定到函数调用的`this`
- 执行此函数代码
- 若函数无返回值，则自动返回这个新对象
```javascript
function fun(){
	this.a = 1
	this.b = 2
}
var instance = new fun()
console.log(instance.a)
```
### 箭头函数
箭头函数的`this`指向就可以理解为传统面向对象语言的`this`啦。它会根据外层的作用域来决定`this`，即取决于外层的函数作用域或全局作用域，且**箭头函数的绑定无法修改**



今天的总结就到这里啦
![在这里插入图片描述](https://0d077ef9e74d8.cdn.sohucs.com/rln2I4a_jpg)