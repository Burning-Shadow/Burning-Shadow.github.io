---
title: 3种方法实现CSS隐藏滚动条并可以滚动内容(转载)
date: 2019-06-08 21:40:00
categories: CSS
tags:
 - webkit-scrollbar
---

帮学长写毕设的时候遇到的一个问题，由于没有什么好的方法所以上网求教并最终找到一篇通俗易懂的博客，码一下以备日后复习所用

<!--more-->

### 方法1：计算滚动条宽度并隐藏起来

在[前端开发博客](https://link.juejin.im?target=http%3A%2F%2Fcaibaojian.com%2F)的侧栏，你可以看到前端日报的那块内容并没有滚动条，但鼠标移上去却可以滚动内容。这是什么技术呢？ 其实我只是把滚动条通过定位把它隐藏了起来。

[演示](https://link.juejin.im?target=http%3A%2F%2Fcaibaojian.com%2Fdemo%2F2018%2F03%2Fscroll.html)

下面给一个简化版的代码

```
<div class="outer-container">
    <div class="inner-container">
    	......
    </div>
</div>
复制代码
.outer-container{
	width: 360px;
	height: 200px;
	position: relative;
	overflow: hidden;
}
.inner-container{
	position: absolute;
	left: 0;
	top: 0;
	right: -17px;
	bottom: 0;
	overflow-x: hidden;
	overflow-y: scroll;
}
复制代码
```

[演示](https://link.juejin.im?target=http%3A%2F%2Fcaibaojian.com%2Fdemo%2F2018%2F03%2Fscroll2.html)

这个代码巧妙的向右移动了17个像素，刚好等于滚动条的宽度。这个值是我手动调试得来的。在chrome和IE没发现问题。

### 方法2：使用三个容器包围起来，不需要计算滚动条的宽度

该代码最早是在[Microsoft](https://link.juejin.im?target=https%3A%2F%2Fblogs.msdn.microsoft.com%2Fkurlak%2F2013%2F11%2F03%2Fhiding-vertical-scrollbars-with-pure-css-in-chrome-ie-6-firefox-opera-and-safari%2F)博客上看到的，跟我上面的思路差不多，只不过人家里面又加多了一个盒子，将内容限制在盒子里面了。这样子就看不到滚动条同时也可以滚动。

代码如下：

```
 <div class="outer-container">
     <div class="inner-container">
        <div class="content">
            ......
        </div>
     </div>
 </div>
复制代码
.element, .outer-container {
  width: 200px;
  height: 200px;
}

.outer-container {
  border: 5px solid purple;
  position: relative;
  overflow: hidden;
}

.inner-container {
  position: absolute;
  left: 0;
  overflow-x: hidden;
  overflow-y: scroll;
}

.inner-container::-webkit-scrollbar {
  display: none;
}
复制代码
```

[演示](https://link.juejin.im?target=http%3A%2F%2Fcaibaojian.com%2Fdemo%2F2018%2F03%2Fscroll3.html)

### 方法3：css隐藏滚动条

同时该文章还分享了一种通过CSS隐藏滚动条的方法，不过这个方法不兼容IE，做移动端的可以使用。

那就是自定义滚动条的伪对象选择器::-webkit-scrollbar

chrome 和Safari

```
.element::-webkit-scrollbar { width: 0 !important }
复制代码
```

IE 10+

```
.element { -ms-overflow-style: none; }
复制代码
```

Firefox

```
.element { overflow: -moz-scrollbars-none; }
```

### 原文链接

作者：前端开发博客

链接：https://juejin.im/post/5aad26ec6fb9a028ba1f416b

来源：掘金

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



> 最后再把这一系列的问题贴上来吧。有些考虑兼容性，有些考虑移动端，暂时本文已经可以满足需要，其他文章日后再一一拜读吧~
>
> [问题链接]([https://juejin.im/search?query=%E9%9A%90%E8%97%8F%E6%BB%9A%E5%8A%A8%E6%9D%A1&type=all](https://juejin.im/search?query=隐藏滚动条&type=all))

