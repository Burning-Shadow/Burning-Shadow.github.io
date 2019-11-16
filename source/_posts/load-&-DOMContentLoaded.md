---
title: load & DOMContentLoaded
date: 2019-04-01 12:24:00
categories: Browser
tags:
 - load
 - DOMContentLoaded
 - DOM渲染
 - cssom渲染
---

> `Load`事件触发代表页面中的 DOM，CSS，JS，图片已经全部加载完毕。

> `DOMContentLoaded`事件触发代表初始的 HTML 被完全加载和解析，不需要等待 CSS，JS，图片加载。

<!--more-->

### 概念

#### 下载/加载

浏览器将资源下载到本地的过程。

#### 解析

将一个元素通过一定的方式转换成另一种形式。

如`html`的解析。

html 下载到浏览器的表现形式就是 包含字符串的文件。浏览器将 html 文件里面的字符串读取到内存中，按照 html 规则，对字符串进行取词编译，将字符串转化成另一种易于表达的数据结构。



**比如对于一段`html`代码，浏览器加载后会执行以下几步操作**

- 对此`html`文件进行编译，转化成 DOM 树的结构
  - ![](https://pic.superbed.cn/item/5ca18ad53a213b0417804952)
- 浏览器会对转化后的数据结构自上而下进行分析：
  - 首先开启下载线程，对所有的资源进行优先级排序下载。同时主线程回对文档进行解析
    - 遇到`<script>`标签时，先阻塞后续内容的解析，同时检查该`<script>`是否已经下载下来。若是则执行代码
    - 遇到`<link>`标签时不会阻塞后续内容的解析（比如 DOM 构建）。检查`<link>`资源是否已经下载。若是则构建`cssom`
    - 遇到 DOM 标签时，执行 DOM 构建，将该 DOM 元素添加到文档树中

> 有一点要注意的是，在 body 中第一个 script 资源下载完成之前，浏览器会进行首次渲染，将该 script 标签前面的 DOM 树和 CSSOM 合并成一棵 Render 树，渲染到页面中。**这是页面从白屏到首次渲染的时间节点，比较关键**。

#### DOM构建

将文档中所有 DOM 元素构建成一个树形结构

> DOM构建是自上而下进行的，会受到 js 执行的干扰

#### CSS构建

将文档中的所有 css 资源合并

#### render树

将 DOM 树和 CSS 合并成一颗渲染树。`render`树在合适的时机会被渲染到页面中（比如遇到`script`，或该`script`还未被下载到本地）

### HTML文档的加载与页面的首次渲染

当我们输入一个页面地址时，发生了如下事情

- 浏览器首先下载该地址所对应的`html`页面
- 浏览器解析`html`页面的 DOM 结构
- 开启下载线程对文档中的所有资源按优先级排序下载
- 主线程继续解析文档，到达`head`节点。（里边无非是外链样式表和外链`js`）
  - 若是外链`js`，则停止解析后续内容，等待资源下载，下载完毕立即执行。
  - 若是外链`css`，则继续解析后续内容
- 解析到`body`
  - 只有 DOM 元素
    - DOM 树构建玩，页面首次渲染
  - 有 DOM 元素、外链`js`
    - 当解析到外链`js`的时候，该`js`尚未下载到本地，则`js`之前的 DOM 会被渲染到页面上，同时`js`会阻止后面 DOM 的构建（即后面的 DOM 节点并不会添加到文档的 DOM 树中），所以在`js`执行完毕之前我们在页面上是看不到该`js`后面的DOM元素。
  - 有 DOM 元素、外链`css`
    - 外链`css`不会影响`css`后面的 DOM 构建，但是会阻碍渲染。简单来说，外链`css`加载完之前，页面还是白屏
  - 有 DOM 元素、外链`js`、外链`css`
    - 外链`js`和外链`css`会影响页面渲染。当`body`中`js`之前的外链`css`未加载完之前，页面是不会被渲染的
    - 当`body`中`js`之前的外链`css`加载完之后，`js`之前的 DOM 树和`css`合并为渲染树，页面渲染出该`js`之前的 DOM 结构
- 文档解析完毕，页面重新渲染。当页面引用所有的`js`同步代码执行完毕，触发`DOMContentLoaded`事件
- `html`文档中的图片资源，`js`代码中有异步加载的`css`、`js`、图片资源都加载完毕之后，`load`事件触发
- 测试代码如下

```html
<body>
  <!-- 白屏 -->
  <div id="div1"></div>
  <!-- 白屏 -->
  <link rel="stylesheet" href="./c1.css" />
  <!-- 白屏 -->
  <link rel="stylesheet" href="./c3.css" />
  <!-- 如果此时 j1.js 尚未下载到本地，则首次渲染，此时的 DOM 树 只有 div1 ，所以页面上只会显示 div1，样式是 c1.css 和 c3.css 的并集。-->
  <!-- 如果此时 j1.js 已经下载到本地，则先执行 j1.js，页面不会渲染，所以此时仍然是白屏。-->
  <!--下面的 js 阻塞了 DOM 树的构建，所以下面的 div2 没有在文档的 DOM 树中。 -->
  <script src="http://test.com:9000/mine/load/case2/j1.js
  "></script>
  <!-- j1.js 执行完毕，继续 DOM 解析，div2 被构建在文档 DOM 树中，此时页面上有了div2 元素，样式仍然是 c1.css 和 c3.css 的并集 -->
  <link rel="stylesheet" href="./c4.css" />
  <!-- c4.css 加载完毕，重新构建render树，样式变成了 c1.css、c3.css 和 c4.css 的并集 -->
  <div id="div2"></div>
  <script>
  // 利用 performance 统计 load 加载时间。
    window.onload=function(){console.log(performance.timing.loadEventStart - performance.timing.fetchStart);}
  </script>
</body>
```

### head 中资源的加载

- `head`中`js`资源加载都会停止后面 DOM 的构建，但是不影响后面资源的下载。
- `css`资源不会阻碍后面 DOM 的构建，但是会阻碍页面的首次渲染。

### body 中资源的加载

- `body`中`js`资源加载都会停止后面 DOM 的构建，但是不影响后面资源的下载。
- `css`资源不会阻碍后面 DOM 的构建，但是会阻碍页面的首次渲染。

### DOMContentLoaded 事件的触发

上面只是讲了 html 文档的加载与渲染，并没有讲 DOMContentLoaded 事件的触发时机。直截了当地结论是，DOMContentLoaded 事件在 **html文档加载完毕，并且 html 所引用的内联 js、以及外链 js 的同步代码都执行完毕后触发**。

### load 事件的触发

当页面 DOM 结构中的`js`、`css`、图片，以及`js`异步加载的`js`、`css` 、图片都加载完成之后，才会触发`load`事件。

注意：

- 页面中引用的js 代码如果有异步加载的`js`、`css`、图片，是会影响`load`事件触发的。
- `video`、`audio`、`flash`不会影响`load`事件触发。





本文转自掘金`lucefer`

[原文地址](https://juejin.im/post/5b2a508ae51d4558de5bd5d1)

