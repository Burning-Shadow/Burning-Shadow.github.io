---
title: HTML5中的WebSocket
categories: HTML5
tags: 
 - HTML5
 - WebSocket
---

我们传统的网络协议，比如`http`、`tcp`等等，都是请求-响应式协议，而若是服务器希望给浏览器发送信息时的渠道则非常有限，大部分只能通过轮询的方式解决。而如果在通信交互频繁的领域，如网游、直播、聊天等领域轮询就显得有些捉襟见肘了，费时费力不说还会浪费大量服务器资源，大部分的精力全部用在了每隔几秒的应答上。

如此一种大佬们采取了一些新的方案：**轮询**和**`Comet`**。

- `Comet`是轮询的一种改进，分为**长轮询**和**流技术**
  - 长轮询是在普通轮询的基础上链接完成后，当服务端未更新数据时链接会保持一段时间的周期，直至**数据状态改变**或**连接时间到**。但缺点也很明显，也不太适用于变化迅速的信息处理服务器，且长轮询更占带宽和资源
  - 流技术则是客户端通过隐藏窗口向服务器发送一个长链接请求，服务器接到请求后作出回应并更新状态，保证前后端连接不过其。但是缺点还是占用带宽、资源，对服务器压力太大了

此时 H5 则新增的`WebSocket`应运而生。

<!--more-->

> `WebSocket`：此协议是为了解决基于浏览器的程序需要拉取资源时必须发起多个 HTTP 请求和长时间的轮询问题。

`WebSocket`借助现有的 HTTP 协议和服务器建立连接，之后双方通过 TCP 进行数据交换（其本质就是 TCP），以此保证数据的实时性和稳定性，实现浏览器与服务器之间的**全双工通信**。

### 特点

- 握手阶段采用 HTTP 协议
- 建立在 TCP 协议基础之上，和 HTTP 协议同属于应用层
- 可以发送文本，亦可以发送二进制数据
- 无同源限制，客户端可以与任意服务器通信
- 协议标识符为`ws`。若加密则为`wss`。

![](https://pic.superbed.cn/item/5ca78a103a213b0417bb6475)

![](https://pic.superbed.cn/item/5ca78ee23a213b0417bb8531)

### 方法

作为一个前端我们就来说说客户端的使用方法咯

```javascript
const ws = new WebSocket('ws://localhost:8080')
```

由于`WebSocket`是纯事件驱动，所以我们可以通过监听事件处理到来的数据和改变的连接状态。

#### open

> 当`open`事件触发时，意味着握手阶段已结束。服务端已处理了连接的请求，可以准备收发数据

服务端相应`WebSocket`连接请求就会触发`open`事件。`onopen`是响应的回调函数

```javascript
ws.onopen = (e)=>{
    console.log('Connection success')
    ws.send(`Hello ${e}`)
}
```

#### Message

> 受到服务器数据会触发消息事件，`onmessage`是响应的回调函数

```javascript
ws.onmessage = (e)=>{
    const data = e.data
    if(typeof data === 'string'){
        console.log('Received string message ', data)
    }else if(data instanceof Blob){
        console.log('Received blob message ', data)
    }
}
```

#### Error

> 发生错误会触发`error`事件，`onerror`是响应的回调函数，会导致连接关闭

```javascript
ws.onerror = (e)=>{
    console.log('WebSocket Error: ', e)
    handleErrors(e)
}
```

#### Close

> `close`方法用来关闭连接。调用`close`方法后将不能发送数据。`close`方法可以传入两个可选参数，`code`和`reason`，以告诉服务端为什么终止连接

```javascript
ws.close(1000, 'Closing normally')
```

#### 补充

整体流程就是这个样子的

```javascript
// 创建一个socket实例：
const socket = new WebSocket(ws://localhost:9093')
// 打开socket
socket.onopen = (event) => {
    // 发送一个初始化消息
  	socket.send('Hello Server!')
  	 // 服务器有响应数据触发
    socket.onmessage = (event) => { 
        console.log('Client received a message',event)
    }
    // 出错时触发，并且会关闭连接。这时可以根据错误信息进行按需处理
    socket.onerror = (event) => {
  	    console.log('error')
    }
    // 监听Socket的关闭
    socket.onclose = (event) => { 
        console.log('Client notified socket has closed',event)
    }
    // 关闭Socket
    socket.close(1000, 'closing normally') 
 }
```

`WebSocket`中的`send`方法只能发送三类数据： UTF-8 的`string`、`ArrayBuffer`和`Blob`。

### 属性

#### readyState

> `WebSocket.CONNECTING`：连接正在进行
>
> `WebSocket.OPEN`：连接已建立
>
> `WebSocket.CLOSING`：连接正在进行关闭握手
>
> `WebSocket.CLOSED`：连接已关闭

这东西就像是`XMLHttpRequest`一样，不知道你们有没有感觉到

#### bufferedAmount

客户端传输大量数据时浏览器会缓存将要流出的数据。

这东西可以判断有多少字节的二进制数据还未发送出去，发送是否结束。

```javascript
ws.onopen = ()=>{
    setInterval(()=>{
        // 缓存未满时发送
        if(ws.bufferedAmount < 1024*5){
            ws.send(data)
        }
    })
}
```

#### protocol

`protocol`代表客户端使用的`WebSocket`协议。（若握手未成功则这个属性是空）

我们可以看一下`WebSocket`实例和服务器连接时握手请求的报文：

```
客户端到服务端：
GET / HTTP/1.1
Connection:Upgrade
Host:127.0.0.1:8088
Origin:null
Sec-WebSocket-Extensions:x-webkit-deflate-frame
Sec-WebSocket-Key:puVOuWb7rel6z2AVZBKnfw==
Sec-WebSocket-Version:13
Upgrade:websocket

服务端到客户端：
HTTP/1.1 101 Switching Protocols
Connection:Upgrade
Server:beetle websocket server
Upgrade:WebSocket
date: Thu, 10 May 2018 07:32:25 GMT
Access-Control-Allow-Credentials:true
Access-Control-Allow-Headers:content-type
Sec-WebSocket-Accept:FCKgUr8c7OsDsLFeJTWrJw6WO8Q=

```

我们可以看出来这其实是一个基于 HTTP 的握手请求，他与 HTTP 不同的是增加了一些头信息

- `Upgrade`字段: 通知服务器，现在要使用一个升级版协议 - `Websocket`。
- `Sec-WebSocket-Key`: 是一个`Base64`编码的值，这个是浏览器随机生成,通知服务器，需要验证下是否可以进行`Websocket`通信
- `Sec_WebSocket-Protocol`: 是用户自定义的字符串，用来标识服务所需要的协议
- `Sec-WebSocket-Version`: 通知服务器所使用的协议版本

当握手成功后，这个时候TCP连接就已经建立了，客户端与服务端就能够直接通过WebSocket直接进行数据传递。不过服务端还需要判断一次数据请求是什么时候开始的和什么时候是请求的结束的。在WebSocket中，由于浏览端和服务端已经打好招呼，如我发送的内容为utf-8 编码，如果我发送0x00,表示包的开始，如果发送了0xFF，就表示包的结束了。这就解决了黏包的问题。



