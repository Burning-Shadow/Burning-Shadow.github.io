---
title: JavaScript中的事件循环&消息队列
categories: JavaScript —— 原理篇
tags: 
- 事件循环
- Event Loop
- 宏任务&微任务
- 执行栈&消息队列
     
---

作为一门设计初衷为了处理浏览器网页交互（`DOM`操作、`UI`动画等）的语言，`JavaScript`只能被设计为单线程（否则多个线程同时处理`DOM`那将会造成混乱）。

可是写过`JavaScript`代码的人都用过定时器、`ajax`、事件绑定等。如果是单线程那岂不是无法完成这些异步请求？

<!--more-->

### 前言

其实，`JavaScript`单线程指的是浏览器中负责解释和执行`JavaScript`代码的只有一个线程——**`JavaScript`引擎线程。**除了他之外浏览器还有其他四个线程：

- 事件触发线程
- 定时器触发线程
- 异步`http`请求线程
- `GUI`渲染线程

当遇到计时器、DOM事件监听或是网络请求时，`JS`引擎会将其交给`webapi`，也就是浏览器提供的相应线程。而JS引擎则继续后边的其他任务，以此方式实现**异步非阻塞。**



在此咱们不得不讲一下`setTimeout`（`setInterval`）函数。这东西没咱们想象的那么准，原因就在于当事件结束后他会将相应的回调函数（`callback`）交还给 **消息队列**。而消息队列中排列着其他的任务，只有轮到它才会被执行。**所以`setTimeout`只能保证其在`ms`毫秒 <u>之后</u> 执行。**



### 事件循环与消息队列

#### 什么是消息队列

众所周知`JavaScript`中的存储区域分为堆区、栈区、还有消息队列区。

- **堆区存放用户创建的对象**。（内训泄露定位的主要区域也在这里。）
- **栈区则是用来处理函数执行（所以又称为执行栈）**。每嵌套一层向栈中推入函数信息，得到返回值后出栈。主代码块**依次**进入执行栈，依次执行。
- **而消息队列则是用来处理异步任务**。每当出现异步调用事件时都会将其入队，<u>执行完毕后</u>再由**任务队列**通知主线程，让`JS`引擎接管此事件。



#### 什么是事件循环

当执行栈为空时（JS引擎线程空闲），事件触发线程会从消息队列中取出一个任务（异步的回调函数）放入执行栈中执行。执行完毕后执行栈再次为空，事件触发线程会重复上一步操作，继续从消息队列中取出一个任务。此机制被称为事件循环（`event loop`）机制。



我们不妨举个例子来看看其操作流程：

```javascript
console.log('script start')

setTimeout(() => {
  console.log('timer 1 over')
}, 1000)

setTimeout(() => {
  console.log('timer 2 over')
}, 0)

console.log('script end')

// script start
// script end
// timer 2 over
// timer 1 over
```

本例中我们先顺序执行同步代码，从消息队列中依次进入执行栈执行，依次打印`script start`，`script end`。

而两个`setTimeout`作为异步代码，分别由定时器触发线程进行监控，时间到后再将其推入消息队列中。

当函数执行栈空时从消息队列中取任务执行。由于"`timer 2 over`"先入队所以先被取出，`timer 1 over`同理。

### 宏任务&微任务

上边这些个机制事实上在ES5已经够用了。但是ES6会有一些问题。因为其引入了新的异步机制——`Promise`

```javascript
console.log('script start')

setTimeout(function() {
    console.log('timer over')
}, 0)

Promise.resolve().then(function() {
    console.log('promise1')
}).then(function() {
    console.log('promise2')
})

console.log('script end')

// script start
// script end
// promise1
// promise2
// timer over
```

为什么`promise1`和`promise2`在“`timer over`”之前打印？

这里就要引入宏任务（`macrotask`）和微任务（`microotask`）。

- 上面提到的一切事件（同步代码块、`setTimeout`、`setInterval`等）都是宏任务
- 而微任务则是`Promise`和`process.nextTick`



顺序的话还是同步事件优先级最高，而这些异步事件其次**。机制同样是事件循环**。

当执行宏任务时遇到`Promise`等，会创建微任务（`.then()`里的回调），并加入微任务队列队尾。

microtask必然是在某个宏任务执行的时候创建的，而在下一个宏任务开始之前，浏览器会对页面重新渲染(`task` >> `渲染` >> `下一个task`(从任务队列中取一个))。同时，在上一个宏任务执行完成后，渲染页面之前，会执行当前微任务队列中的所有微任务。



> 在某一个`macrotask`执行完后，在重新渲染与开始下一个宏任务之前，就会将在它执行期间产生的所有microtask都执行完毕（在渲染前）。

>在node环境下，`process.nextTick`的优先级高于`Promise`，也就是说：在宏任务结束后会先执行微任务队列中的`nextTickQueue`，然后才会执行微任务中的`Promise`。

### Node 中的 Event Loop

先看一张图

![](https://pic.superbed.cn/item/5cb161123a213b041729d3ff)

如图我们可得到以下几点信息：

- 我们的`js`代码（`APPLICATION`）会先进入 V8 引擎。V8 引擎中主要是一些`setTimeout`之类的方法
- 其次若我们的代码中执行了`node API`，比如`require('fs').read()`，`node`就会交给`libuv`库处理。这个`libuv`库就是`node`的事件环
- `libuv`库是通过单线程异步的方式来处理事件。我们可以看到`work threads`是个多线程的队列，通过外面`event loop`阻塞的方式来进行异步调用
- 等到`work threads`队列中有执行完成的事件，就会通过`EXECUTE CALLBACK`回调给`EVENT QUEUE`队列，把他放入队列中
- 最后通过事件驱动的方式，取出`EVENT QUEUE`队列的事件，交给我们应用

Node 的 Event loop 分为6个阶段，它们会按照顺序反复运行

![](https://pic.superbed.cn/item/5cb162a23a213b041729e708)

#### node-EventLoop-Status

##### timer

- timers 阶段会执行 `setTimeout` 和 `setInterval`。一个 `timer` 指定的时间并不是准确时间，而是在达到这个时间后尽快执行回调，可能会因为系统正在执行别的事务而延迟。下限的时间有一个范围：`[1, 2147483647]` ，如果设定的时间不在这个范围，将被设置为1.

##### I/O

- I/O 阶段会执行除了 close 事件，定时器和 `setImmediate` 的回调

##### idle, prepare

- idle, prepare 阶段内部实现

##### poll

- poll 阶段很重要，这一阶段中，系统会做两件事情
  1. 执行到点的定时器
  2. 执行 poll 队列中的事件
- 并且当 poll 中没有定时器的情况下，会发现以下两件事情
  - 如果 poll 队列不为空，会遍历回调队列并同步执行，直到队列为空或者系统限制
  - 如果 poll 队列为空，会有两件事发生 
    - 如果有 `setImmediate` 需要执行，poll 阶段会停止并且进入到 check 阶段执行 `setImmediate`
    - 如果没有 `setImmediate` 需要执行，会等待回调被加入到队列中并立即执行回调
    - 如果有别的定时器需要被执行，会回到 timer 阶段执行回调。

##### check

- check 阶段执行 `setImmediate`

##### close callbacks

- close callbacks 阶段执行 close 事件。并且在 Node 中，有些情况下的定时器执行顺序是随机的

- ```javascript
  setTimeout(() => {
      console.log('setTimeout');
  }, 0);
  setImmediate(() => {
      console.log('setImmediate');
  })
  // 这里可能会输出 setTimeout，setImmediate
  // 可能也会相反的输出，这取决于性能
  // 因为可能进入 event loop 用了不到 1 毫秒，这时候会执行 setImmediate
  // 否则会执行 setTimeout
  ```

- 当然在这种情况下，执行顺序是相同的

- ```javascript
  var fs = require('fs')
  
  fs.readFile(__filename, () => {
      setTimeout(() => {
          console.log('timeout');
      }, 0);
      setImmediate(() => {
          console.log('immediate');
      });
  });
  // 因为 readFile 的回调在 poll 中执行
  // 发现有 setImmediate ，所以会立即跳到 check 阶段执行回调
  // 再去 timer 阶段执行 setTimeout
  // 所以以上输出一定是 setImmediate，setTimeout
  ```

- 上面介绍的是`macrotask`的执行情况。被创建的`microtask`会在以上宏任务完成后立即执行

- ```javascript
  setTimeout(()=>{
      console.log('timer1')
  
      Promise.resolve().then(function() {
          console.log('promise1')
      })
  }, 0)
  
  setTimeout(()=>{
      console.log('timer2')
  
      Promise.resolve().then(function() {
          console.log('promise2')
      })
  }, 0)
  
  // 以上代码在浏览器和 node 中打印情况是不同的
  // 浏览器中一定打印 timer1, promise1, timer2, promise2
  // node 中可能打印 timer1, timer2, promise1, promise2
  // 也可能打印 timer1, promise1, timer2, promise2
  ```

> Node 中的 `process.nextTick` 会先于其他 microtask 执行。

### 你真的懂了吗老弟？

#### 嵌套

先给个简单的，关于宏任务和微任务之间的嵌套

```javascript
Promise.resolve().then(()=>{
  console.log('Promise1')  
  setTimeout(()=>{
    console.log('setTimeout2')
  },0)
})

setTimeout(()=>{
  console.log('setTimeout1')
  Promise.resolve().then(()=>{
    console.log('Promise2')    
  })
},0)
```

打印结果是`Promise1  setTimeout1  Promise2  setTimeout2`

**解析**

- 一开始执行栈的同步任务执行完毕，回去**微任务队列**找
- 清空微任务队列，输出`Promise1`，同时生成一个异步任务`setTimeout1`
- 去宏任务队列查看此时队列是`setTimeout1`在`setTimeout2`之前，因为`setTimeout1`执行栈一开始的时候就开始异步执行，所以输出`setTimeout1`。在执行`setTimeout1`时会生成`Promise2`的一个微任务，放入**微任务队列**中
- 接着又是一个循环，去**清空微任务队列**，输出`Promise2`
- 清空完**微任务队列**，就又去**宏任务队列**中取一个。这次取的是`setTimeout2`

#### node环境下的嵌套+setTimeout+setImmediate+nextTick

##### 引子

首先我们说一下这仨有啥区别

> `setTimeout`采用的是类似 IO 观察者。精度不高，可能有延迟执行的情况发生。动用了红黑树所以消耗资源大
>
> `setImmediate`采用的是`check`观察者。消耗资源小，也不会造成阻塞，但效率最低
>
> `process.nextTick`采用的是`idle`观察者。效率最高，消费资源小，但会阻塞 CPU 的后续调用
>
> 三种观察者的优先级顺序是：**idle观察者 >>  IO观察者 > check观察者**

前两者都会进入等待队列，而`process.nextTick`是一个比较特殊的存在

比如下面的代码

```javascript
A();
process.nextTick(B);
C();
setImmediate(D);
```

会有如下的顺序

![](https://pic.superbed.cn/item/5cb173b93a213b04172a851d)

**所以`nextTick`的优先级要高于前两者（`setTimeout`和`setImmediate`一样，都是进入等待队列，所以我就不写setImmediate了啊）**

```javascript
setTimeout(function(){
    console.log("setTimeout");
},0);
 
setImmediate(function(){
    console.log("setImmediate");
});
```

但是这段代码的打印结果不确定，但是`setTimeout`在前的概率更大些，因为 IO 观察者的优先级要大于`check`观察者

##### 例子

![](https://pic.superbed.cn/item/5cb162a23a213b041729e708)

为了防止大家绕晕所以我们先把上面的`node event loop`那张图拿下来方便查看.

下面上代码

```javascript
setImmediate(()=>{
  console.log('setImmediate1')
  setTimeout(()=>{
    console.log('setTimeout1')    
  },0)
})
setTimeout(()=>{
  console.log('setTimeout2') 
  process.nextTick(()=>{console.log('nextTick1')})
  setImmediate(()=>{
    console.log('setImmediate2')
  })   
},0)
```

答案有两种：

```
setImmediate1,setTimeout2,setTimeout1,nextTick1,setImmediate2
setImmediate1,setTimeout2,nextTick1,setImmediate2,setTimeout1
```

- 首先我们可以看到上面的代码先执行的是`setImmediate1`,此时`event loop`在**check队列**

- 然后`setImmediate1`从队列取出之后，输出`setImmediate1`，然后会将`setTimeout1`执行

- 此时`event loop`执行完**check队列**之后，开始往下移动，接下来执行的是**timers队列**

- 这里会有问题，我们都知道`setTimeout`设置延迟为0的话，其实还是有`4ms`的延迟。那么这里就会有两种情况。

  - **第一种，`setTimeout1`已经执行完毕**：
    - 根据node事件环的规则，我们会执行完所有的事件，即取出**timers队列**中的`setTimeout2,setTimeout1`
    - 此时根据队列先进先出规则，输出顺序为`setTimeout2,setTimeout1`，在取出`setTimeout2`时，会将一个`process.nextTick`执行（执行完了就会被放入**微任务队列**），再将一个`setImmediate`执行（执行完了就会被放入**check队列**）
    - 到这一步，`event loop`会再去寻找下个事件队列，此时`event loop`会发现**微任务队列**有事件`process.nextTick`，就会去清空它，输出`nextTick1`
    - 最后`event loop`找到下个有事件的队列**check队列**，执行`setImmediate`，输出`setImmediate2`
  - **第二种，`setTimeout1`还未执行完毕**
    - 此时`event loop`找到**timers队列**，取出**timers队列**中的`setTimeout2`，输出`setTimeout2`，把`process.nextTick`执行，再把`setImmediate`执行
    - 然后`event loop`需要去找下一个事件队列，**这里大家要注意一下**，这里会发生2步操作，
      - 1、**`setTimeout1`执行完了，放入`timers`队列**。
      - 2、找到微任务队列清空。
      - 所以此时会先输出`nextTick1`
    - 接下来`event loop`会找到**`check`队列**，取出里面已经执行完的`setImmediate2`
    - 最后`event loop`找到**`timers`队列**，取出执行完的`setTimeout1`。**这种情况下event loop比上面要多切换一次**

  

如果你把这个也搞懂了，那大概就真的懂`Event Loop`了8~。