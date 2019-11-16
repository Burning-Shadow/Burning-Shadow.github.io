---
title: 从Event Loop 到 Vue 中的 nextTick
date: 2019-03-27 21:00:10
categories: Vue
tags: 
 - Vue
 - nextTick
 - Event Loop
---

记得咱讲微任务的时候提到了最常用的两个创建微任务 API ：`Promise`和`nextTick`。`Promise`咱们已经讲过了，所以今天一起来看看`nextTick`

<!--more-->

### 前言

`nextTick`是 Vue 中一个特殊的 API 。一般用作 DOM 更新完毕之后执行一个回调。（`vm.$nextTick([callback])`）

#### 用法

将回调延迟到下次 DOM 更新循环之后执行。在修改数据之后立即使用它，然后等待 DOM 更新。它跟全局方法`Vue.nextTick`一样，不同的是回调的`this`自动绑定到调用它的实例上。

```javascript
new Vue({
    el: '#main',
    data: {
        list: [
            'AAAAAAAAAA',
            'BBBBBBBBBB',
            'CCCCCCCCCC'
        ]
    },
    mounted: function () {
        this.list.push('DDDDD')
    }
})
```

我们更该数据之后，`list`中新加入的数据`DDDDD`会立刻呈现出来。但是我们打印他们 DOM 结构的`length`时候会显示`3`

```javascript
mounted: function () {
    this.list.push('DDDDD')
    console.log(this.$el.querySelectorAll('.item').length)  // 3
}
```

此时使用 Vue 中的`nextTick`就可以解决问题

```javascript
mounted: function () {
    this.list.push('DDDDD')
    Vue.nextTick(function() {
        console.log(this.$el.querySelectorAll('.item').length)  // 4
        // ... 计算
    })
}
```

>当你设置 vm.someData = 'new value' ，该组件不会立即重新渲染。当刷新队列时，组件会在事件循环队列清空时的下一个“tick”更新。多数情况我们不需要关心这个过程，但是如果你想在 DOM 状态更新后做点什么，这就可能会有些棘手。虽然 Vue.js 通常鼓励开发人员沿着“数据驱动”的方式思考，避免直接接触 DOM，但是有时我们确实要这么做。为了在数据变化之后等待 Vue 完成更新 DOM ，可以在数据变化之后立即使用 Vue.nextTick(callback) 。这样回调函数在 DOM 更新完成后就会调用。



### 从一个栗子说起

```javascript
new Promise((resolve) => {
    console.log(1);				// 同步
    
    process.nextTick(() => {
    	console.log(2);
    });
    
    resolve();
    
    setTimeout(() => {
    	console.log(6);
	}, 0);
    
    process.nextTick(() => {
    	console.log(3);
    });
    
    console.log(4);				// 同步
}).then(() => {
    console.log(5);
});

console.log(7);					// 同步
```

上边的代码输出结果是`1、4、7、2、3、5、6`。同样是创建微任务的宏任务，凭什么你`nextTick`就是比我`setTimeout`先执行？

![](<https://0d077ef9e74d8.cdn.sohucs.com/rlVj7im_jpg>)

#### 宏任务 & 微任务

一个宿主环境只有一个时间循环，但可以有多个任务队列。（`macrotask & microtask`）。每次事件循环的时候，都会先执行红任务队列中的任务，再执行微任务队列中的任务。

> 宏任务：`script`、`setTimeout`、`setInterval`、`setImmediate(只有IE支持)`、`I/O`、`UI rendering`
>
> 微任务：`process.nextTick`、`Promise`、`Object.observer`、`MutationObserver`



微任务都会被添加到**当前循环的微任务队列**之中。所以会比当前循环中的所有宏任务要后执行，会比下个循环中的宏任务要先执行。

### 单拎出来`process.nextTick`和`Promise`

```javascript
process.nextTick(() => {
    console.log(1); 
});
new Promise((resolve) => {
    resolve();
}).then(() => {
    console.log(2);
});
process.nextTick(() => {
    console.log(3); 
});
```

结果打印了`1、3、2`

![](https://pic.superbed.cn/item/5c9b75503a213b0417379f1d)

在`node`环境下，`_tickCallback`在每一次执行完`TaskQueue`中的一个任务后被调用，而这个`_tickCallback`中实质上干了两件事

- 执行完所有`nextTickQueue`中的任务
- 执行`_runMicrotasks`函数，执行`microtask`中的部分（`promise.then()`注册的回调）。所以很明显优先级`process.nextTick > promise.then`。

### setImmediate？？？

```javascript
setImmediate(() => {
    console.log(1);
});

setTimeout(() => {
    console.log(2);
}, 0);
```

这东西跑的结果好像不太稳定，有时候打印1，有时候打印2。



`nodejs`官网给出的解释是：

- `setImmediate()`: 是被设计用来一旦当前阶段的任务执行完后执行。
- `setTimeout()`: 是让代码延迟执行。

**如果没有在一个I/O周期执行，那么其执行顺序是不确定的。**

**如果在一个I/O周期执行，`setImmediate`总是优先于`setTimeout`执行。**

```javascript
const fs = require('fs');

fs.readFile(__filename, () => {
    setTimeout(() => {
        console.log('timeout');
    }, 0);
    setImmediate(() => {
        console.log('immediate');
    });
});
```

如果这样的话那么就总是先打印`immediate`再打印`timeout`啦。

### 贴段源码

```javascript
export const nextTick = (function () {
  const callbacks = []
  let pending = false
  let timerFunc

  function nextTickHandler () {
    pending = false
    const copies = callbacks.slice(0)
    callbacks.length = 0
    for (let i = 0; i < copies.length; i++) {
      copies[i]()
    }
  }

  //other code
})()
```

callbacks就是缓存的所有回调函数，nextTickHandler就是实际调用回调函数的地方。

```javascript
if (typeof Promise !== 'undefined' && isNative(Promise)) {
    var p = Promise.resolve()
    var logError = err => { console.error(err) }
    timerFunc = () => {
        p.then(nextTickHandler).catch(logError)
        if (isIOS) setTimeout(noop)
    }
} else if (typeof MutationObserver !== 'undefined' && (
    isNative(MutationObserver) ||
    MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
    var counter = 1
    var observer = new MutationObserver(nextTickHandler)
    var textNode = document.createTextNode(String(counter))
    observer.observe(textNode, {
        characterData: true
    })
    timerFunc = () => {
        counter = (counter + 1) % 2
        textNode.data = String(counter)
    }
} else {
    timeFunc = () => {
        setTimeout(nextTickHandle, 0)
    }
}
```

为让这个回调函数延迟执行，vue优先用promise来实现，其次是html5的MutationObserver，然后是setTimeout。前两者属于microtask，后一个属于macrotask。下面来看最后一部分

```javascript
return function queueNextTick(cb?: Function, ctx?: Object) {
    let _resolve
    callbacks.push(() => {
        if (cb) cb.call(ctx)
        if (_resolve) _resolve(ctx)
    })
    if (!pending) {
        pending = true
        timerFunc()
    }
    if (!cb && typeof Promise !== 'undefined') {
        return new Promise(resolve => {
            _resolve = resolve
        })
    }
}
```

这就是我们真正调用的nextTick函数，在一个event loop内它会将调用nextTick的cb回调函数都放入callbacks中，pending用于判断是否有队列正在执行回调，例如有可能在nextTick中还有一个nextTick，此时就应该属于下一个循环了。最后几行代码是promise化，可以将nextTick按照promise方式去书写（暂且用的较少）。