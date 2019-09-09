---
title: JavaScript中如何实现一个相对准确的计时器
categories: JavaScript —— 原理篇
tags: 
 - setTimeout
---

由于JavaScript是一门单线程语言，所以`setTimeout`的误差无法完全被解决

也许会因为浏览器中的事件过多导致该宏时间等待时间增长

也许会因为也许是因为回调中的某些问题使得时延增加。

还可能是因为`HTML5`标准中规定函数第二个参数不得小于`4ms`而强制增加。

所以，页面开久了定时器会不准

<!--more-->

所以我们可以手动实现一个相对较准的计时器

```javascript
var period = 60 * 1000 * 60 * 2				// periodms之后
var startTime = new Date().getTime();
var count = 0								// 第count个任务
var end = new Date().getTime() + period
var interval = 1000							// 间隔
var currentInterval = interval

function loop() {
  count++
  var offset = new Date().getTime() - (startTime + count * interval); // 代码执行所消耗的时间
  var diff = end - new Date().getTime()
  
  var h = Math.floor(diff / (60 * 1000 * 60))
  var hdiff = diff % (60 * 1000 * 60)
  var m = Math.floor(hdiff / (60 * 1000))
  var mdiff = hdiff % (60 * 1000)
  var s = mdiff / (1000)
  var sCeil = Math.ceil(s)					// s向上取整
  var sFloor = Math.floor(s)				// s向下取整
  
  currentInterval = interval - offset // 得到下一次循环所消耗的时间
  console.log('时：'+h, '分：'+m, '毫秒：'+s, '秒向上取整：'+sCeil, '代码执行时间：'+offset, '下次循环间隔'+currentInterval) // 打印 时 分 秒 代码执行时间 下次循环间隔

  setTimeout(loop, currentInterval)
}

setTimeout(loop, currentInterval)
```

