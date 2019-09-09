---
title: 如何实现一个Event Bus
categories: JavaScript —— 原理篇
tags:
 - Event Bus实现
---

面试官在对 Vue 或者 React 进行深入询问的时候一定会问到组件间通讯。除去基本的父子组件和中大型项目所用到的，就剩下`Event Bus`了。

这时候面试官问你：老弟你有没有想过自己实现一个`Event Bus`？

<!--more-->

### 提要

安卓中的`Event Bus`、NodeJS 中的`Event`模块（）等等均需要用到`Event Bus`。所以理解其中原理也是一个必不可少的过程和要求。

具体代码我们可以参考 Node 中的 [Event](<http://nodejs.cn/api/events.html>) API，它就是**发布订阅模式**的典型应用

### Event Bus

这东西如上所述，是咱们在小型项目中用来通知下兄弟组件执行一些方法的。那么咱们具体应该如何实现呢？

如果希望了解`EventBus`作为中间件通信可以点击这里，[《Vue自定义组件事件传递：EventBus部分》](<https://juejin.im/post/5b2f2402f265da59b37e7ab1>)。

如果希望了解 Vue 父子组件间通讯和组件内通讯则可以看我相关的博客 [《Vue中的$emit、$on和v-on》]([https://burning-shadow.github.io/2019/03/22/Vue%E4%B8%AD%E7%9A%84$emit%E3%80%81$on%E5%92%8Cv-on/](https://burning-shadow.github.io/2019/03/22/Vue中的$emit、$on和v-on/))



但是既然咱们开了这个口还是**总结**一下叭：

`EventBus`的作用就是作为一个中间件，是链接两个组件的一座桥梁。

- 发送方通过`EventBusName.$emit('eventName', data)`将数据和事件名传递给`EventBus`
- 接收方则通过`EventBusName.$on('eventName', methods)`对数据进行处理

### 实现

通过刚才的大致介绍我们得知：`EventBus`可以理解为一个**发布订阅模式**的典型应用，所以说如果让我们实现，那么就可以理解为实现一个双向的发布订阅器。

#### 初版

##### 初始化

我们通过 ES6 的`Class`关键字对`Event`进行初始化。包括`Event`的事件清单和监听者上限。

我们选择了`Map`作为储存事件的结构，因为作为键值对的存储方式`Map`比一般对象更加适合，操作更简洁

```javascript
class EventEmeitter {
  constructor() {
    this._events = this._events || new Map(); // 储存事件/回调键值对
    this._maxListeners = this._maxListeners || 10; // 设立监听上限
  }
}
```

##### 监听与触发

触发监听函数我们可以用`apply`和`call`两种方法。Node 中当参数小于 3 个时使用`call`，否则用`apply`。

```javascript

// 触发名为type的事件
EventEmeitter.prototype.emit = function(type, ...args) {
  let handler;
  // 从储存事件键值对的this._events中获取对应事件回调函数
  handler = this._events.get(type);
  if (args.length > 0) {
    handler.apply(this, args);
  } else {
    handler.call(this);
  }
  return true;
};

// 监听名为type的事件
EventEmeitter.prototype.addListener = function(type, fn) {
  // 将type事件以及对应的fn函数放入this._events中储存
  if (!this._events.get(type)) {
    this._events.set(type, fn);
  }
};
```

##### 如果有多个监听者呢

```javascript
// 重复监听同一个事件名
emitter.addListener('arson', man => {
  console.log(`expel ${man}`);
});
emitter.addListener('arson', man => {
  console.log(`save ${man}`);
});

emitter.emit('arson', 'low-end'); // expel low-end
```

#### 升级改造

##### 监听器、触发器升级

我们的`addListener`实现方法还不够健全。在绑定第一个监听者之后就无法对后续监听者进行绑定。故我们需要将所有监听者放入一个数组中

```javascript

// 触发名为type的事件
EventEmeitter.prototype.emit = function(type, ...args) {
  let handler;
  handler = this._events.get(type);
  if (Array.isArray(handler)) {
    // 如果是一个数组说明有多个监听者,需要依次此触发里面的函数
    for (let i = 0; i < handler.length; i++) {
      if (args.length > 0) {
        handler[i].apply(this, args);
      } else {
        handler[i].call(this);
      }
    }
  } else { // 单个函数的情况我们直接触发即可
    if (args.length > 0) {
      handler.apply(this, args);
    } else {
      handler.call(this);
    }
  }

  return true;
};

// 监听名为type的事件
EventEmeitter.prototype.addListener = function(type, fn) {
  const handler = this._events.get(type); // 获取对应事件名称的函数清单
  if (!handler) {
    this._events.set(type, fn);
  } else if (handler && typeof handler === 'function') {
    // 如果handler是函数说明只有一个监听者
    this._events.set(type, [handler, fn]); // 多个监听者我们需要用数组储存
  } else {
    handler.push(fn); // 已经有多个监听者,那么直接往数组里push函数即可
  }
};

```

如此即可触发多个监听者的函数了

```javascript
// 监听同一个事件名
emitter.addListener('arson', man => {
  console.log(`expel ${man}`);
});
emitter.addListener('arson', man => {
  console.log(`save ${man}`);
});

emitter.addListener('arson', man => {
  console.log(`kill ${man}`);
});

// 触发事件
emitter.emit('arson', 'low-end');
//expel low-end
//save low-end
//kill low-end

```

##### 移除监听

我们通过`removeListener`函数移除监听函数，但无法移除匿名函数

```javascript
EventEmeitter.prototype.removeListener = function(type, fn) {
  const handler = this._events.get(type); // 获取对应事件名称的函数清单

  // 如果是函数,说明只被监听了一次
  if (handler && typeof handler === 'function') {
    this._events.delete(type, fn);
  } else {
    let postion;
    // 如果handler是数组,说明被监听多次要找到对应的函数
    for (let i = 0; i < handler.length; i++) {
      if (handler[i] === fn) {
        postion = i;
      } else {
        postion = -1;
      }
    }
    // 如果找到匹配的函数,从数组中清除
    if (postion !== -1) {
      // 找到数组对应的位置,直接清除此回调
      handler.splice(postion, 1);
      // 如果清除后只有一个函数,那么取消数组,以函数形式保存
      if (handler.length === 1) {
        this._events.set(type, handler[0]);
      }
    } else {
      return this;
    }
  }
};

```

#### 仍有问题

我们已经基本完成了`Event`最重要的几个方法,也完成了升级改造,可以说一个`Event`的骨架是被我们开发出来了,但是它仍然有不足和需要补充的地方.

> 1. 鲁棒性不足: 我们没有对参数进行充分的判断,没有完善的报错机制.
> 2. 模拟不够充分: 除了`removeAllListeners`这些方法没有实现以外,例如监听时间后会触发`newListener`事件,我们也没有实现,另外最开始的监听者上限我们也没有利用到.

当然,这在面试中现场写一个Event已经是很够意思了,主要是体现出来对**发布-订阅**模式的理解,以及针对多个监听状况下的处理,不可能现场撸几百行写一个完整Event.

索性[Event](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FGozala%2Fevents%2Fblob%2Fmaster%2Fevents.js)库帮我们实现了完整的特性,整个代码量有300多行,很适合阅读,你可以花十分钟的时间通读一下,见识一下完整的Event实现.

### 原文

作者：寻找海蓝96

链接：https://juejin.im/post/5ac2fb886fb9a028b86e328c

### 总结

`Event Bus`的实现其实和 Vue 实现双向绑定的方式一样，把这个拿出来其实我是有炒冷饭的嫌疑的。不过作为一个冷门的知识点拿出来单说其实也不错，至少不会被一下问懵。今后如果能想起来的话我会着重复习并总结一下 JS 中常用的设计模式以及 OOP 的实现方式等，那么，后会有期江湖见！