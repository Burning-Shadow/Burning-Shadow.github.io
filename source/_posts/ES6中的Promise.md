---
title: ES6中的Promise
categories: ES6
tags: 
- Promise
- ES6
---

### 提要

为解决传统异步所造成的回调地狱，社区提出了`Promise`方案并最终将其写入了语言标准。**其用法如其名——承诺：即非立即兑现，通过`then`方法调用其状态，通过`catch`捕获错误。**

<!--more-->

### 状态

- `Pending`：进行中
- `Fulfilled`：已成功
- `Rejected`：已失败

`Promise`的另一个特性是**无法改变**。正如其英文寓意一样，一旦状态改变就无法继续更改。故我们将`Fulfilled`和`Rejected`统一称为`Resolved`（已定型）

### 缺点

当然`Promise`也有一些缺点：

- 一旦创建立即执行，无法中途取消。
- 若不设置回调函数则其内部错误不会抛向外部
- `Pending`状态时无法知道目前进展到哪一阶段（开始或即将完成）



### 用法

```javascript
var p = new Promise((resolve, reject)=>{
    console.log('Create a Promise')
    resolve('success')
})
console.log('After new Promise')
p.then(val=>{
    console.log(val)
})

// Create a Promise
// After new Promise
// success
// Promise {<resolved>: undefined}
```

如图，我们创建`promise`时其会立即执行，一旦状态发生改变那么立即调用`then`方法中的函数。

没看出来？咱们加个定时器试试

```javascript
function timeout(ms){
    return new Promise((resolve, reject)=>{
        console.log('Running Immediately')
        setTimeout(resolve, ms, 'done')
    })
}
timeout(100).then(val=>{
	console.log(val)
},err=>{
    console.log(err)
})
// Running Immediately
// Promise {<pending>}
// done
```

看到了吧，只要状态一旦改变那么会立即执行`then`方法。我们平日所写的`ajax`函数从此也再不需要进行嵌套回调了

```javascript
var getJSON = (url)=>{
    var promise = new Promise((resolve, reject)=>{
        var client = new XMLHttpRequest()
        client.open('GET', url)
        client.onreadystatechange = handler
        client.responseType = 'json'
        client.setRequestHeader('Accept', 'application/json')
        client.send()
        
        function handler(){
            if(this.readyState !== 4){
                return;
            }
            if(this.status === 200){
                resolve(this.response)
            }else{
                reject(new Error(this.statusText))
            }
        }
    })
    return promise
}
```

看，我们只需要将`ajax`函数包装并返回`promise`，就可以得到其状态了。调用更简单，`then`方法随时接收`promise`所返回的`resolve`或`reject`

```javascript
getJSON('/admin/getIndentify').then( json => {
    console.log('json has already resolved,the data is:\n')
    console.log(json)
}, err => {
    console.log(err)
})
```

#### 状态传递

如果是`promise`之间相互调用那么被调用者的状态会决定调用者的状态

```javascript
var p1 = new Promise((resolve, reject)=>{
    setTimeout( () => {
      reject(new Error('fail'))  
    }, 3000)
})
var p2 = new Promise((resolve, reject) => {
    setTimeout( () => {
      resolve(p1)  
    }, 4000)
})

p2.then( result => {
    console.log(result)
}).then( err => {
    console.log(err)
})

// Promise {<pending>}
// （3s之后）
// Uncaught (in promise) Error: fail
//     at setTimeout (<anonymous>:3:14)
```

3s后 p1 状态为`reject`，p2 在1s之后改变，`resolve`返回的时 p1。**由于 p2 返回的是另一个`Promise`，导致 p2 的状态无效，所以 p1 的状态决定了 p2 的状态。**故后面的`then`语句都变成了针对 p1 的。最后通过`then`抛出错误。

Ps：上面的错误是在3s之后抛出，因为得到 p1 状态后 p2 就不再执行，自然那 4s 也就不用等了

#### then()

**`then`方法的作用是为`Promise`实例添加状态改变时的回调函数。**

`Promise.prototype.then()`方法返回的是一个新的`Promise`实例，故可以采用链式写法。

通过链式调用我们就可以完成步骤顺序调用

```javascript
getJSON('/admin/getUrl').then((data)=>{
    getJSON(data.commentURL)
}).then((comments)=>{
    console.log(comments)
},(err)=>{
    console.log(err)
})
```

#### catch()

**`catch`方法用于指定发生错误时的回调函数**，是`.then(null, rejection)`的简写

> `reject`方法的作用等同于抛出错误。若`Promise`状态已变为`Resolved`，再抛出错误是无效的

```javascript
var p = new Promise( (resolve, reject) => {
    resolve('ok')
    throw new Error('test')
})

p.then(val => {
    console.log(val)
}).catch(err => {
    console.log(err)
})

// ok
// 之后的错误并没有被抛出，因为在其创建之前promise状态就已改变
```

其他不再赘述，我们重点说一下其具体用法：**冒泡性质**

`Promise`对象的错误具有冒泡性质，会一直想后传递，直到被捕获为止。即，错误总是会被下一个`catch`捕获。

```javascript
getJSON('/admin/getUrl').then( data =>{
    getJSON(data.commentURL)
}).then( comments =>{
    console.log(comments)
}).catch( err =>{
    // 处理前面3个Promise产生的错误
})
```

一般来说不要在`then`方法中定义`Rejected`状态的回调参数，利用`catch`方法可以捕获所有之前所产生的错误

但是注意，**一定要记得写`catch`。否则`Promise`对象抛出的错误不会传递到外层代码**

##### 自定义方法之done()

无论`Promise`对象的回调链以`then`还是`catch`解为，只要最后一个方法抛出错误，都有可能无法捕捉到（`Promise`内部的错误不会冒泡到全局）。故我们提供一个`done`方法，它总是处于回调链的尾端，保证抛出任何可能出现的错误

```javascript
asyncFunc()
	.then(f1)
	.catch(r1)
	.then(f2)
	.done()
```

其实现如下

```javascript
Promise.prototype.done = function(onFulfilled, onRejected){
    this.then(onFulfilled, onRejected)
        .catch(function(resson){
        // 抛出一个全局错误
        setTimeout( ()=>{ throw reason}, 0)
    })
}
```

##### 自定义方法之finally()

不论`Promise`对象最后状态如何都会执行的操作我们放在`finally`中。比如使用`finally`关掉服务器

```javascript
server.listen(8080)
  .then( () => {
    // run test
  })
  .finally(server.stop)
```

**其实现如下**

```javascript
Promise.prototype.finally = function(callback){
    let P = this.constructor
    return this.then(
    	val => P.resolve(callback()).then(() => val),
        reason => P.resolve(callback()).then(() => {throw reason})
    )
}
```



#### Promise.all()

将多个`Promise`实例包装成一个新的`Promise`实例，参数不一定为数组，只要有`Iterator`接口且返回的每个成员都是`promise`即可。

当数组中**所有**实例状态都已改变时新实例状态才会跟着改变

```javascript
var promise = [2,3,5,7,11,13].map(id=>{
    return getJSON('admin/' + id + '.json')
})

Promise.all(promise).then(posts=>{
    // ....
}).catch(err=>{
    // ....
})
```

当上面6个`promise`的状态都变为`fulfilled`或其中一个变为`rejected`才会调用`Promise.all()`后的函数

#### Promise.race()

与上面的不同，但凡有一个参数状态改变，实例状态就会跟着改变。**那个率先改变的参数的返回值就是传递给实例的参数**

### Promise.resolve()

**通过`Promise.resolve()`方法可以将现有对象转换为`Promise`对象**

```javascript
Promise.resolve('foo')
new Promise( resolve => {
    resolve('foo')
})
```

当然，参数可能是多种形式。对各种形式的参数处理方式不同

#### 参数为Promise实例

不做修改，直接返回

#### 参数为thenable对象

> `thenable`对象是具有`then`方法的对象

`Promise.resolve`方法会将此对象转为`Promise`对象，然后立即执行`thenable`对象的`then`方法

#### 参数是原始值或不具then方法

`Promise.resolve`方法会直接返回一个新的，状态为`fulfilled`的`Promise`对象

#### 无参

直接返回一个状态为`fulfilled`的对象

### Promise.reject()

生成一个状态为`rejected`的`Promise`对象的实例，