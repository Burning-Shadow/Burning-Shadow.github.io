---
title: HTML5中的新API
date: 2019-04-06 12:24:01
categories: HTML5
tags: 
 - Canvas
 - Audio
 - Viedo
 - Geolocation
 - WebSockets
 - Web Storage
 - Communication
 - Web Workers
 - requestAnimationFrame
---

H5 中新兴的一些API让我们可以更方便地操作数据和浏览器。此文作为一个提纲，日后还会继续更新。

<!--more-->

### Canvas

`Canvas`可以理解为一块画布，在上面可以完成对所想图形的绘制

```html
<canvas id="diagonal" style="border:1px solid;" width="200" height="200"></canvas>
```

```javascript
function drawDiagonal(){
     //取的canvas元素及其绘图上下文
     var canvas = document.getElementById(‘diagonal’);
     var context = canvas.getContext('2d');
     // 用绝对坐标来创建一条路径
     context.beginPath();
     context.moveTo(70,140);
     context.lineTo(140,70);
     // 将这条线绘制到canvas上
     context.stroke();
}
window.addEventListener('load',drawDiagonal,true);
```

### Audio && Video

H5中的多媒体支持，比如现在的播放器都是H5播放器，再无需下载插件了

```html
<audio id="clickSound">
    <source src="a.ogg">
    <source src="b.mp3">
</audio>
<button id="toggle" onclick="toggleSound()">play</button>
```

```javascript
function toggleSound() {
    var music = document.getElementById('clickSound');
    var toggle = document.getElementById('toggle');
    if(music.paused){
        music.play();
        toggle.innerHTML = 'Pause';
    }else{
        music.pause();
        toggle.innerHTML = 'Play';
    }
}
```

### Geolocation

请求一个位置信息，若用户同意则浏览器会返回其位置信息。该信息是通过支持 H5 地理定位功能的底层设备提供给浏览器的，位置由维度/经度坐标和一些其他的数据组成。

```javascript
window.addEventListener('load',loadDemo,true);
function loadDemo(){
    if(navigator.geolocation){
        navigator.geolocation.watchPosition(
            updateLocation,
            handleLocationError,
            {maximumAge:20000}
        );
    }
}
function updateLocation(position){
    var latitude = position.coords.latitude;
    var longitude = position.coords.longitude;
    var accuracy = position.coords.accuracy;
    var timestamp = position.timestamp;
}
```

### WebSockets

作为一种新型的 H5 双全工协议，它实现了浏览器和服务器之间的双向通讯。

在WebSocket API 中，浏览器和服务器只需要一个握手的动作，之后通过其简历的通道完成双向数据传输，代替了之前复杂而耗力的轮询

```javascript
var  wsServer = 'ws://localhost:8888/Demo';
var  websocket = new WebSocket(wsServer);
websocket.onopen = function (evt) { onOpen(evt) };
websocket.onclose = function (evt) { onClose(evt) };
websocket.onmessage = function (evt) { onMessage(evt) };
websocket.onerror = function (evt) { onError(evt) };
function onOpen(evt) {
  console.log("Connected to WebSocket server.");
}
function onClose(evt) {
  console.log("Disconnected");
}
function onMessage(evt) {
  console.log('Retrieved data from server: ' + evt.data);
}
function onError(evt) {
  console.log('Error occured: ' + evt.data);
}
```

### Web storage

#### 提要

`web storage`就是`localStorage`和`sessionStorage`的统称，至于区别嘛只在于生命周期。`localStorage`存储在本地，生命周期是永久，而`sessionStorage`虽然也存储在本地，但是生命周期仅仅为当前窗口或标签页，一旦页面被关闭则其存储的数据亦会丢失。

> Ps：页面及标签页仅指顶级窗口。若一个标签页包含多个`iframe`则其亦属于同源页面，之间共享`sessionStorage`

#### 离线缓存

> HTML5 的 Web storage API 采用了离线缓存，会生成一个清单文件（manifest file)，这个清单文件实质就是一系列的URL列表文件，这些URL分别指向页面当中的html,css,javascript,图片等相关内容。当使用离线应用时，应用会引入这一清单文件，浏览器会读取这一文件，下载相应的文件，**并将其缓存到本地**。使得这些web应用能够脱离网络使用，而**用户在离线时的更改也同样会映射到清单文件中，并在重新连线之后将更改返回应用**，工作方式与我们现在所使用的网盘有着异曲同工之处。

我们需要在页面头加入`manifest`属性

```html
<!DOCTYPE HTML>
<html manifest = "cache.manifest">
...
</html>
```

然后`cache.manifest`文件的书写方式为

```
CACHE MANIFEST
#v0.11
CACHE:
js/app.js
css/style.css
NETWORK:
resourse/logo.png
FALLBACK:
/ /offline.html
```

离线存储的manifest一般由三个部分组成:

1. CACHE:表示需要离线存储的资源列表，由于包含manifest文件的页面将被自动离线存储，所以不需要把页面自身也列出来。
2. NETWORK:表示在它下面列出来的资源只有在在线的情况下才能访问，他们不会被离线存储，所以在离线情况下无法使用这些资源。不过，如果在CACHE和NETWORK中有一个相同的资源，那么这个资源还是会被离线存储，也就是说CACHE的优先级更高。
3. FALLBACK:表示如果访问第一个资源失败，那么就使用第二个资源来替换他，比如上面这个文件表示的就是如果访问根目录下任何一个资源失败了，那么就去访问offline.html。

### Communication

>出于安全方面的考虑，运行在同一浏览器中的框架，标签页，窗口间的通信一直都受到了严格的限制。
>如果浏览器内部能提供直接的通信机制，就能更好的组织这些应用。
>为了满足上述需求，浏览器厂商和标准制定机构一致同意引入一种新功能：跨文档消息通信。

跨文档消息通信可以确保`iframe`，标签页，窗口间安全的进行跨院通信。它把`postMessage` 的API 定义为发送消息的标准方式。

发送数据的方式非常简单

```javascript
chatFrame.contentWindow.postMessage(‘Hello,world’,’http://jartto.wang');
```

而接收消息时仅需要在页面中添加一个事件处理函数。消息到达时通过检查消息的来源来决定是否对此消息进行处理

```javascript
window.addEventListener('message',messageHandler,true);
function messageHandler(e){
    switch(e.origin) {
        case 'friend.example.com':
            // 处理消息
            processMessage(e.data);
            break;
        default:
            // 消息来源无法识别，消息被忽略
    }
}
```

### Web Workers

>HTML5 Web Workers可以让Web 应用程序具备后台处理能力，它对多线程的支持非常好，因此，使用了HTML5的Javascript应用程序可以充分利用多核CPU带来的优势。将耗时长的任务分配给HTML5 Web Workers执行，可以避免弹出脚本运行缓慢的警告。
>
>Web Workers不能直接访问Web 页面和DOM API。
>
>Web Workers的另一个用途是监听由后台服务器广播的新闻消息，收到后台服务的消息后，将其显示在Web页面上。

### requestAnimationFrame

> 浏览器可以优化并行的动画动作，更合理的重新排列动作序列，并把能够合并的动作放在一个渲染周期内完成，从而呈现出更流畅的动画效果。比如，通过requestAnimationFrame()，JS动画能够和CSS动画/变换或SVG SMIL动画同步发生。另外，如果在一个浏览器标签页里运行一个动画，当这个标签页不可见时，浏览器会暂停它，这会减少CPU，内存的压力，节省电池电量。

```javascript
// shim layer with setTimeout fallback
window.requestAnimFrame = (function(){
  return  window.requestAnimationFrame       ||
          window.webkitRequestAnimationFrame ||
          window.mozRequestAnimationFrame    ||
          function( callback ){
            window.setTimeout(callback, 1000 / 60);
          };
})();
// usage:
// instead of setInterval(render, 16) ....
(function animloop(){
  requestAnimFrame(animloop);
  render();
})();
// place the rAF *before* the render() to assure as close to
// 60fps with the setTimeout fallback.
```



这个是转载`Jartto`大佬的博客，[原文链接](http://jartto.wang/2016/07/25/make-an-inventory-of-html5-api/)在这里，作为一个知识树的构建，今后会在这个栏目中继续更新相关的API