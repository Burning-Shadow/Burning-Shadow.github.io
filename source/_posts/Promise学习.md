---
title: Promise记录
categories: ES6
time: 2020.10.4 21:45:31
---

初学 Promise 时就让我很好奇一点，Promise 是如何将异步转化为同步语法进行执行的？

<!-- more -->

### 观察者模式

Vue 的设计指导模式就是发布-订阅模式，而发布订阅算是观察者模式的衍生产物。

> 观察者模式：发布者和观察者共同维护一个列表，当列表内容发生变化时会以广播的形式通知给每一个**观察者**
>
> 发布订阅模式：发布者维护多个列表，而订阅者可以根据自身需要进行特定属性的订阅

```javascript
/*
 * 观察者模式
 */
class Observer {
  constructor() {
    this.list = [];
  }

  subscribe(observer) {
    this.list.push(observer)
  }

  notifyAll(value) {
    this.list.forEach(observe => observe(value))
  }
}

// 观察者
const observer = new Observer();
observer.subscribe(value => {
  console.log("第一个观察者，接收到的值为:");
  console.log(value)
});
observer.subscribe(value => {
  console.log("第二个观察者，接收到的值为");
  console.log(value)
});

setTimeout(() => {
  observer.notifyAll('hello world');
}, 1000);
```

上述的发布订阅就是顺嘴一提，主角还是观察者模式。

在大家学习的时候不知有没有好奇，Promise 是如何得知异步事件执行完毕的呢？

先想想你平时的使用方式，以 ajax 为例

```javascript
const $ajax =  param => {
  return new Promise((resovle, reject) => {
    var xhr = new XMLHttpRequest();
    xhr.open(param.type || "get", param.url, true);
    xhr.send(param.data || null);

    xhr.onreadystatechange = () => {
     var DONE = 4; // readyState 4 代表已向服务器发送请求
     var OK = 200; // status 200 代表服务器返回成功
     if(xhr.readyState === DONE){
      if(xhr.status === OK){
        resovle(JSON.parse(xhr.responseText));
      } else{
        reject(JSON.parse(xhr.responseText));
      }
     }
    }
  })
}

$ajax(url).then(res => {
  console.log(res);
  // Do something .... 
}).then(res => {
  console.log(res);
  // Do something ....
})
```

其中 then 就是观察者，一旦状态改变则将控制权交由其订阅者。如此一来我们就有了如下对应关系：

- resolve / reject —— notifyAll
- subscribe —— then

### 改版为Promise

有了上述设计模式的支持我们可以尝试手写简易版本，目前我们已知的几个属性如下：

> - 一旦创建会立即执行
> - 一旦开始则无法停止且只执行一次
> - 接收一个成功后的 function
> - 状态改变后无法回退
> - then 对应我们的观察者 subscribe
> - resolve / reject 对应我们的发布者 notifyAll

然后魔改一下上面的观察者模式代码：

```javascript
const STATUS = {
  PENDING: 'pending',
  RESOLVED: 'resolved',
}

class MyPromise {
  constructor(executor) {
    this.list = [];
    this.status = STATUS.PENDING;
    try {
      executor(this.resolve);
    } catch (err) {
      throw Error(err);
    }
  }

  // 原 subscribe
  resolve = (val) => {
    if (this.status === STATUS.PENDING) {
      this.list.forEach(fn => fn(val));
      this.status = STATUS.RESOLVED;
    }
  }

  // 原 notify
  then(observer) {
    this.list.push(observer);
  }
}

const p = new MyPromise(resolve => {
  setTimeout(() => {
    resolve("hello world");
  }, 1000);
});

p.then(value => console.log(value));
```

ok那我们第一步已经完成，魔改成功。

如果你打开 chrome 断点调试你会发现， 最终的结果打印经历了这么几个步骤：

> - 初始化 resolve 函数
> - 初始化监听列表 & 状态（防止执行多次）
> - 异步操作（定时任务）
> - 将结果传入 resolve
> - 改变状态，清空（执行）监听列表

那么是什么时候执行的 push 操作呢？

#### push callback

由于情况较为特殊，断点并未显示执行 then 的过程，所以我们在 push 操作前可以打印一句话作为参考坐标

console.log(observer)

而其执行位置则是在异步操作执行结束后，也就是断点刚刚到 setTimeout 内部回调【resolve('hello world')】时执行了push 操作。

故**完整过程如下**

> - 初始化【resolve、监听列表、状态】
> - 异步操作
> - push callback(then中的回调) to list
> - 结果传入 resolve
> - 改变状态，清空（执行）监听列表

### 完善Promise

上边的内容我们仅仅是作为观察者模式的延展，根据 [Promise/A+](https://promisesaplus.com/) 规范，真正的 promise 我们还需要考虑以下几点

> - 异步执行失败的状态
> - 链式调用

#### 状态

首先就是加上 reject 状态及 value

```javascript
const STATUS = {
  PENDING: 'pending',
  RESOLVED: 'resolved',
  REJECTED: 'rejected',
}

class MyPromise {
  constructor(executor) {
    this.list = [];
    this.status = STATUS.PENDING;
    this.value = undefined;

    try {
      executor(this.resolve);
    } catch (err) {
      throw Error(err);
    }
  }

  resolve = (val) => {
    if (this.status === STATUS.PENDING) {
      this.list.forEach(fn => fn(val));
      this.value = val;
      this.status = STATUS.RESOLVED;
    }
  }

  reject = reason => {
    if (this.status === STATUS.PENDING) {
      this.list.forEach(fn => fn(val));
      this.value = val;
      this.status = STATUS.REJECTED;
    }
  }

  then(observer) {
    console.log('observer === ', observer);
    this.list.push(observer);
  }
}

const p = new MyPromise(resolve => {
  setTimeout(() => {
    resolve("hello world");
  }, 1000);
});

p.then(value => console.log(value));
```

但此时有了不同的状态（成功 & 失败）后，还要对之前的then做一下处理，以适应。链式调用我们就要分情况讨论了

#### 调用

promise 既然接受两个状态回调，那么我们自然也需要更改一下构造器里的list，将其分为监听 resolve 的以及接收 reject 的两种不同状态对应的回调函数。

```javascript
const STATUS = {
  PENDING: 'pending',
  RESOLVED: 'resolved',
  REJECTED: 'rejected',
}

class MyPromise {
  constructor(executor) {
    this.resolveCallbackArr = [];
    this.rejectCallbackArr = [];
    this.status = STATUS.PENDING;
    this.value = undefined;

    try {
      executor(this.resolve);
    } catch (err) {
      throw Error(err);
    }
  }

  resolve = (val) => {
    if (this.status === STATUS.PENDING) {
      this.list.forEach(fn => fn(val));
      this.value = val;
      this.status = STATUS.RESOLVED;
    }
  }

  reject = reason => {
    if (this.status === STATUS.PENDING) {
      this.list.forEach(fn => fn(val));
      this.value = val;
      this.status = STATUS.REJECTED;
    }
  }

  // 区别于最初的变化值，我们需要将 observer 改为 成功 / 失败态监听函数，并将观察者要执行的操作依次 push 进相应的数组中
  then = (onResolve, onReject) => {
  /*
   * 若传入非函数则包装为函数，通过此种写法可以保证值的透传
   * ---------------------------------------------
   * new Promise(resolve=>resolve(8))
   *  .then()
   *  .catch()
   *  .then(function(value) {
   *    alert(value)
   * })
   * 的行为应该和
   * new Promise(resolve=>resolve(8))
   *  .then(val => val)
   *  .catch(val => val)
   *  .then(function(value) {
   *    alert(value)
   * })
   * 是一样的
   */
    onResolve = typeof onResolve === 'function' ? onResolve : val => val;
    onReject = typeof onReject === 'function' ? onReject : reason => reason;

    // 保险策略，防止传入同步函数
    if (this.status === Status.RESOLVE) {
      onResolve(this.value);
    }
    if (this.status === Status.REJECT) {
      onReject(this.value);
    }
    if (this.status === Status.PENDING) {
      // 判断是否 push ，push 进哪里（依赖收集）
      this.resolveCallbackArr.push(succObserver);
      this.resolveCallbackArr.push(failObserver);
    }
  }
}
```

#### 链式调用

首先，**Promise 链式调用的原理即是将结果包装为一个新的 Promise**，那么我们就必须要保证每次返回的均为一个新的 Promise（调用then的均为Promise实例），而 new 操作之后必定为 Promise 对象，所以只需要保证执行完 then 方法后返回的仍然是 Promise 。**如此我们就需要重新定义观察者（`subscribe`） —— then**

```javascript
// 区别于最初的变化值，我们需要将 observer 改为 成功 / 失败态监听函数
then = (onResolve, onReject) => {
  onResolve = typeof onResolve === 'function' ? onResolve : val => val;
  onReject = typeof onReject === 'function' ? onReject : reason => reason;

  return new Promise((resolve, reject) => {
    try {
      if (this.status === STATUS.RESOLVED) {

        const result = onResolve(this.value);
        if (result instanceof MyPromise) {
          result.then(resolve, reject);
        } else {
          resolve(result);
        }
      }

      if (this.status === STATUS.REJECTED) {
        const result = onReject(this.value);
        if (result instanceof MyPromise) {
          result.then(resolve, reject);
        } else {
          reject(result);
        }
      }

      if (this.status === STATUS.PENDING) {
        this.resolveCallbackArr.push(val => {
          const result = onResolve(val);
 
          if (result instanceof MyPromise) {
            result.then(resolve, reject);
          } else {
            resolve(result);
          }
        });

        this.rejectCallbackArr.push(val => {
          const result = onReject(val);
          if (result instanceof MyPromise) {
            result.then(resolve, reject);
          } else {
            reject(result);
          }
        });
      }
    } catch (err) {
      reject(err);
    }
  })
}
```

### catch、finally

> `catch`方法返回一个[Promise](https://developer.mozilla.org/zh-CN/docs/Web/API/Promise)，并且处理拒绝的情况。它的行为与调用[`Promise.prototype.then(undefined, onRejected)`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) 相同作为错误捕获

所以我们有了之前的 then 作为铺垫，这个方法就简单了许多

```javascript
catch = (onReject) => {
  return this.then(undefined, onReject);
}
```

`finally` 避免了同样的语句需要在 [`then()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) 和 [`catch()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch) 中各写一次的情况。

> `finally`的返回结果仍为 [Promise](https://developer.mozilla.org/zh-CN/docs/Web/API/Promise)
>
> `finally`的回调函数中不接收任何参数，它仅用于无论最终结果如何都要执行的情况。
>
> **执行完毕后其返回结果为`finally`所接收的最后一个`value`**

```javascript
finally = (cb) => {
  return this.then(value => {
    cb();
    return value;
  }, err => {
    cb();
    throw err
  });
};
```

### Promise.resolve&reject

这里实现时应当将其写为静态方法，毕竟可以通过类直接调起。所以和我们上面实现的 resolve 方法还有区别，为了便于区分我们将原 resolve 方法转移至 constructor 中。

Promise.resolve 的主要作用在于**将某个对象转化为 Promise 对象**

故接收到的参数有以下几种可能的值：

>- **Promise实例**：不作处理，返回 Promise 实例
>
>- **tenable 对象**：转为 Promise 对象并立即执行该对象的 then 方法后返回
>
>- **基本类型 / 非`tenable`对象 / 无参数**：返回一个新的 resolved 态的 Promise 对象
>

```javascript
static resolve = (p) => {
  if (p instanceof MyPromise) {
    return p.then();
  }
  if (p.then === undefined) {
    return new MyPromise((resolve, reject) => {
      resolve(p);
    });
  }
  return new MyPromise((resolve, reject) => {
    resolve(p).then();
  });
}
```



```javascript
static reject = (p) => {
  if (p instanceof MyPromise) {
    return p.catch();
  }
  if (p.then === undefined) {
    return new MyPromise((resolve, reject) => {
      reject(p);
    });
  }
  return new MyPromise((resolve, reject) => {
    reject(p);
  });
}
```

### 总结

Promise 归根结底就是观察者模式的视线版本，其发展离不开以下几点：

#### 异步

1. **Push**：new Promise，定义回调，then 方法完成 callback 收集，将依赖项 push 进对应 list 中【res push 进 resolveCallbackArr，rej push 进 rejectCallbackArr】。
2. **Run**：回调传入 constructor 执行（若catch 到 err 则通过 reject 抛出）
3. **Wait**：等待状态改变，调用 resolve / reject
4. **ForEach**：resolve、reject更改状态，并执行 callbackList【resolveCallbackArr、rejectCallbackArr】 中的回调
5. Eg：

```javascript
const observer = new Promise((res, rej) => {
  setTimeout(() => {
    res('hello wrold');
  }, 1000)
}).then(res => {
  console.log('回调结束，打印结果');
  console.log(res);
})
```

#### 同步（手动调用resolve）

1. **Change**：用户手动改变状态
2. **Run**：回调（值）传入 constructor 执行
3. **forEach**：直接调用 resolve，执行 callbackList【resolveCallbackArr】 中的回调
4. Eg:

```javascript
const observer = new Promise((res, rej) => {
  res('hello wrold');
}).then(res => {
  console.log('回调结束，打印结果');
  console.log(res);
})
```

### 代码地址

Xxx

### 参考文章

[《从设计模式角度分析Promise：手撕Promise并不难》](https://juejin.im/post/6844903834184073224#heading-10)

[Promise/A+](https://promisesaplus.com/)

[《观察者模式》](https://juejin.im/post/6844904134840156168#comment)

[模拟实现一个 Promise.finally #109](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/109)

[《ECMAScript6入门 - Promise》](https://es6.ruanyifeng.com/)

[剖析Promise内部结构，一步一步实现一个完整的、能通过所有Test case的Promise类#3](https://github.com/xieranmaya/blog/issues/3)

