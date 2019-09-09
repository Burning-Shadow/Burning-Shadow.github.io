---
title: 执行上下文对象Context
categories: JavaScript
tags:
 - context
 - 作用域
 - 执行上下文对象
---

在我们的 API 实现的几篇博客中（apply、call、bind等）有些我们用到了执行上下文对象 `Context`。但这究竟是个什么玩意儿？好像平时也没有注意过，所以今天我们就从其生成过程到使用方式详细的说一下此物件的用法。

<!--more-->

### 预编译阶段

代码无论在何处 `js` 引擎都会在执行前对代码进行**预编译**。其过程分为以下四步：

- 解析 `js` 代码
- 建立 `arguments` 对象——`context` + 参数
- 建立作用域链
- 确定 `this` 指向

欸，这就到咱们的主题咯。这个 `arguments`类数组对象之前咱们见过，都具有一个默认的参数，我们可以对其进行拆分。

举一个例子，我们的 `call` 函数的实现

```javascript
Function.prototype.mycall = function (context) {
  console.log("function's context = ", context)
  console.log("function's this = ", this)
  console.log("function's args = ", arguments)

  if (typeof this !== 'function') {
    throw new TypeError('not funciton')
  }
  context = context || window
  context.fn = this
  let arg = [...arguments].slice(1)
  let result = context.fn(...arg)
  delete context.fn
  return result
} 
```

测验代码也很简单，直接复制到控制台就可以。

```javascript
var a = {
 b: 1,
 c: {
  d: 2
 }
}
function fun(c, d){
 console.log(c);
 console.log(d);
 console.log(this.b)
}
fun.mycall(a, 8, 9);
```

比较懒的小伙伴直接看结果：

![myapply运行结果](https://ae01.alicdn.com/kf/HTB1mNRNaYY1gK0jSZTE760DQVXaS.png)

瞅着没，一应俱全。

- `this`被默认绑定到调用函数；
- `context` 则是函数的执行位置
- `arguments`对象就比较有意思了。他的第一个参数为`context`，其后则是函数传入的参数

这次大概懂了吧。那么我们再来细究一下他的原理

### 执行上下文

顾名思义，**函数执行时的上下文**，也可以理解为**其执行的环境**。

它具有三个属性：

- 变量对象（`variable object`）
- 作用域链（`scope chain`）
- `this`指针

 js 在执行过程中会有一个上下文栈，上下文栈中存放的就是不同的上下文对象。故而，**当前执行代码的`context`对象总是在栈顶**

#### 变量对象

变量对象是`context`对象中的一个重要属性。其创建过程如下

- 创建`arguments`对象，其中保有多个属性，其`key`值为`0、1、2、3...`而`value`值就是传入的参数的实际值。
- 在`arguments`中找到第一个参数（即`context`对象，其施展的作用域）
- 找到作用域中的所有**声明**
  - 若为`function`则属性名为函数名，属性值为函数的引用
  - 若为`var`则属性名即为变量名，属性值视情况而定（可能为`undefined`）

如此，我们在绑定函数作用域时即可从其执行环境（也就是`context`对象中）找到相应的变量对象。

#### 变量提升

知道了变量对象和执行环境之间的关系，我们可以联系一下之前的知识：变量提升。

这东西想必大家还记得，js 引擎在对变量或函数进行处理前会先将函数声明即变量提升至顶端，而在赋值的时机则是看程序员的意思。

那么函数声明和变量提升哪个在前边儿？

```javascript
f();

var f = function(){
    var a = 1;
    console.log(a)
}

function f(){
    var b = 2;
    console.log(b);
}

f();
```

打印结果为 `2,1` 所以显然是`function`的声明在前~

这也和咱们编译阶段的`arguments`创建类似咯，对作用域中的变量进行预声明，进而方便在执行时通过作用域链查找使用，非常方便的呢。

对这部分有兴趣的 boy 可以自行上网百度作用域链，当然也可以选择查看我的另一篇博客[《JavaScript中的闭包》]([https://burning-shadow.github.io/2019/03/20/JavaScript%E4%B8%AD%E7%9A%84%E9%97%AD%E5%8C%85/](https://burning-shadow.github.io/2019/03/20/JavaScript中的闭包/))，其中有关于变量查询的方式，涉及到一部分作用域链的知识。还有一篇写的不太好的关于作用域的文章[《JavaScript中的作用域》]([https://burning-shadow.github.io/2019/03/20/JavaScript%E4%B8%AD%E7%9A%84%E4%BD%9C%E7%94%A8%E5%9F%9F/](https://burning-shadow.github.io/2019/03/20/JavaScript中的作用域/))，欢迎来踩。

今天到此为止，告辞。

