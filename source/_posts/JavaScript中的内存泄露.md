---
title: JavaScript中的内存泄漏
date: 2019-03-30 21:00:20
categories: JavaScript
tags:
 - 内存泄漏
---

### 定义

`js`如同其他高级语言一样都有垃圾回收机制，会周期性的检查之前分配的内存是否可达，帮助开发者管理内存。对不可达的内存通过算法确定、标记并适时回收。

而内存泄露则可以理解为当应用程序不再需要占用内存时，由于某些原因操作系统未回收其内存。

<!--more-->

### 堆？栈？队列？

#### 任务队列

由于`js`是单线程，所以所有的任务都需要排队。学过操作系统的都知道单线程最大的浪费就是输入输出时的等待，所以`JavaScript`设计者采取了另一种策略：主线程忽略`IO`设备，挂起处于等待中的任务，先运行它后面的任务。等到`IO`设备返回结果后再讲挂起任务继续执行下去

##### 同步任务（synchronous）

> 在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务

##### 异步任务（asynchronous）

> 不进入主线程、而进入"**任务队列**"（task queue）的任务，只有"**任务队列**"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

#### 函数执行栈
![在这里插入图片描述](https://pic.superbed.cn/item/5c93bcca3a213b0417da3c67)

函数的嵌套调用就是通过函数执行栈。每嵌套一层向栈中推入函数信息，得到返回值后出栈

#### 堆

而主要的用户创建的对象就存放在堆中。**内存泄漏定位的主要区域就在这里**。

### 垃圾回收

V8 实现了准确式 GC，GC 算法采用了分代式垃圾回收机制。因此，V8 将内存（堆）分为新生代和老生代两部分。

#### 新生代算法

新生代中的对象一般存活时间较短，使用**`Scavenge GC`**算法。

在新生代空间中，内存空间分为两部分，分别为 From 空间和 To 空间。在这两个空间中，必定有一个空间是使用的，另一个空间是空闲的。新分配的对象会被放入 From 空间中，当 From 空间被占满时，新生代 GC 就会启动了。**算法会检查 From 空间中存活的对象并复制到 To 空间中，如果有失活的对象就会销毁**。当复制完成后将 From 空间和 To 空间**互换**，这样 GC 就结束了。

#### 老生代算法

老生代中的对象一般存活时间较长且数量也多，使用了两个算法，分别是标记清除算法和标记压缩算法。

在讲算法前，先来说下什么情况下对象会出现在老生代空间中：

- 新生代中的对象是否已经经历过一次 Scavenge 算法，如果经历过的话，会将对象从新生代空间移到老生代空间中。
- To 空间的对象占比大小超过 25 %。在这种情况下，为了不影响到内存分配，会将对象从新生代空间移到老生代空间中。

老生代中的空间很复杂，有如下几个空间

```c++
enum AllocationSpace {
  // TODO(v8:7464): Actually map this space's memory as read-only.
  RO_SPACE,    // 不变的对象空间
  NEW_SPACE,   // 新生代用于 GC 复制算法的空间
  OLD_SPACE,   // 老生代常驻对象空间
  CODE_SPACE,  // 老生代代码对象空间
  MAP_SPACE,   // 老生代 map 对象
  LO_SPACE,    // 老生代大空间对象
  NEW_LO_SPACE,  // 新生代大空间对象

  FIRST_SPACE = RO_SPACE,
  LAST_SPACE = NEW_LO_SPACE,
  FIRST_GROWABLE_PAGED_SPACE = OLD_SPACE,
  LAST_GROWABLE_PAGED_SPACE = MAP_SPACE
};
```

##### Mark-and-Sweep（标记清除）算法



在老生代中，以下情况会先启动`mark-and-sweep`。

- 某一个空间没有分块的时候
- 空间中被对象超过一定限制
- 空间不能保证新生代中的对象移动到老生代中

>在这个阶段中，会遍历堆中所有的对象，然后标记活的对象，在标记完成后，销毁所有没有被标记的对象。在标记大型对内存时，可能需要几百毫秒才能完成一次标记。这就会导致一些性能上的问题。为了解决这个问题，2011 年，V8 从 stop-the-world 标记切换到增量标志。在增量标记期间，GC 将标记工作分解为更小的模块，可以让 JS 应用逻辑在模块间隙执行一会，从而不至于让应用出现停顿情况。但在 2018 年，GC 技术又有了一个重大突破，这项技术名为并发标记。该技术可以让 GC 扫描和标记对象时，同时允许 JS 运行

标记清除算法由以下几部分组成

- 垃圾回收器创建了一个“`roots`”列表。`Roots`通常是代码中全局变量的引用。`JavaScript` 中,“`window`”对象是一个全局变量，被当作`root `。`window`对象总是存在，因此垃圾回收器可以检查它和它的所有子对象是否存在（即不是垃圾）；
- 所有的 `roots` 被检查和标记为激活（即不是垃圾）。所有的子对象也被递归地检查。从`root`开始的所有对象如果是可达的，它就不被当作垃圾。
- 所有未被标记的内存会被当做垃圾，收集器现在可以释放内存，归还给操作系统了。

![在这里插入图片描述](https://pic.superbed.cn/item/5c93bcee3a213b0417da3dae)

> 不需要的引用是指开发者明知内存引用不再需要，却由于某些原因，它仍被留在激活的`root`树中。在`JavaScript`中，不需要的引用是保留在代码中的变量，它不再需要，却指向一块本该被释放的内存。

##### 标记压缩算法

> 清除对象后会造成堆内存出现碎片的情况，当碎片超过一定限制后会启动压缩算法。在压缩过程中，将活的对象像一端移动，直到所有对象都移动完成然后清理掉不需要的内存。

### 举几个栗子

#### 意外的全局变量

##### 粗心

如果定义时忘记了`var`那么引擎会自动帮你创建一个全局变量。虽然无伤大雅还是尽量避免的好

##### this创建的意外

```javascript
function foo() { 
    this.a = "accidental global"; 
}
```

#### 闭包&&定时器

```javascript
var theThing = null; 
var replaceThing = function () { 
  var originalThing = theThing; 
  var unused = function () { 
    if (originalThing) {		// 引用了originalThing，阻止了originalThing的回收
      console.log("hi"); 
    }
  }; 
 
  theThing = { 
    longStr: new Array(1000000).join('*'), 		// 逐渐堆积，令originalThing体量不断增大
    someMethod: function () { 
      console.log(someMessage); 
    } 
  }; 
}; 
 
setInterval(replaceThing, 1000);
```

代码片段做了一件事情：每次调用`replaceThing`，`theThing`得到一个包含一个大数组和一个新闭包（`someMethod`）的新对象。同时，变量`unused`是一个引用`originalThing`的闭包（先前的`replaceThing`又调用了`theThing`）。思绪混乱了吗？

> 一旦一个作用域被创建为闭包，那么它的父作用域将被共享

最重要的事情是，**闭包的作用域一旦创建，它们有同样的父级作用域，作用域是共享的**。`someMethod()`可以通过`theThing`使用，`someMethod()`与`unused`分享闭包作用域。**`unused`引用了`originalThing`,这阻止了`originalThing`的回收，尽管`unused`不会被使用,但是`someMethod`依然可以通过`theThing`来访问`replaceThing`作用域外的变量**。

当这段代码反复运行，就会看到内存占用不断上升，垃圾回收器（`GC`）并无法降低内存占用。本质上，闭包的链表已经创建，每一个闭包作用域携带一个指向大数组的间接的引用，造成严重的内存泄露。

当然，我们可以通过在`replaceThing`的最后添加`originalThing = null`来修复此问题。

#### DOM引用

当需要删除DOM的引用时候注意一点：同一份DOM一般拥有两份引用：DOM树和字典

```javascript
var elements = {
    button: document.getElementById('button'),
    image: document.getElementById('image')
};
function doStuff() {
    elements.image.src = 'http://example.com/image_name.png';
}
function removeImage() {
    // The image is a direct child of the body element.
    document.body.removeChild(document.getElementById('image'));
    // At this point, we still have a reference to #button in the
    //global elements object. In other words, the button element is
    //still in memory and cannot be collected by the GC.
    elements.img = null		// tell the engine to collect the memory
}
```

还有一个额外的考虑，当涉及 DOM 树内部或叶子节点的引用时，必须考虑这一点。假设你在 JavaScript 代码中保留了对 table 特定单元格（`<td>`）的引用。有一天，你决定从 DOM 中删除该 table，但扔保留着对该单元格的引用。直观地来看，可以假设 GC 将收集除了该单元格之外所有的内容。实际上，这不会发生的：该单元格是该 table 的子节点，并且 children 保持着对它们 parents 的引用。也就是说，在 JavaScript 代码中对单元格的引用会导致整个表都保留在内存中的。保留 DOM 元素的引用时，需要仔细考虑。

好啦今天的话题就讲到这里
![在这里插入图片描述](https://0d077ef9e74d8.cdn.sohucs.com/rln2I4a_jpg)