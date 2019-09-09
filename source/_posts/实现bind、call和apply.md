---
title: 实现bind、call和apply
categories: JavaScript —— 原理篇
tags: 
 - 实现原理
 - bind
 - call
 - apply
---

对`call`、`bind`和`apply`三个库函数的实现

<!--more-->

### call

```javascript
Function.prototype.myCall = function (context) {
  var context = context || window
  // 给 context 添加一个属性
  // getValue.call(a, 'yck', '24') => a.fn = getValue
  context.fn = this
  // 将 context 后面的参数取出来
  var args = [...arguments].slice(1)
  // getValue.call(a, 'yck', '24') => a.fn('yck', '24')
  var result = context.fn(...args)
  // 删除 fn
  delete context.fn
  return result
}
```

### apply

```javascript
Function.prototype.myApply = function (context) {
  var context = context || window
  context.fn = this

  var result
  // 需要判断是否存储第二个参数
  // 如果存在，就将第二个参数展开
  if (arguments[1]) {
    result = context.fn(...arguments[1])
  } else {
    result = context.fn()
  }

  delete context.fn
  return result
}
```

### bind

- 不传入第一个参数则默认为window
- 改变了`this`指向，让新的对象可以执行该函数。那么思路是否可以变成给新的对象添加一个函数，然后再执行完后删除。

```javascript
Function.prototype.myBind = function (context) {
  // 调用者必须是 function
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  var _this = this
  
  // 第一个参数是context
  var args = context ? [...arguments].slice(1) : [...arguments]
  
  // 返回一个函数
  return function F() {
    // 因为返回了一个函数，我们可以 new F()，所以需要判断
    if (this instanceof F) {
      return new _this(...args, ...arguments)
    }
    return _this.apply(context, args.concat(...arguments))
  }
}
```

