---
title: CSS中的rem
date: 2019-03-24 21:00:16
categories: CSS
tags:
 - css
 - rem
 - 布局
---

近些年来随着微信小程序的蓬勃发展，原本适用于PC端的字体型号px好像不再那么的万金油了，尤其是面对不同的手机屏幕，px的适配性显得过于呆板。之前的一些大厂选用了通用型号的屏幕适配，多余的部分则用留白来填补。这样虽然让Plus型号得以施展，但是一段留白总归看的诸位土豪心中不爽。随着CSS的发展，rem横空出世解决了这一难题

为了大家的时间，我们采用我初中答政治题的方式，分“是什么”，“为什么”，“怎样做”三个方面给大家讲一下rem的详情，希望最大程度的节省大家的时间

<!--more-->

#### 是什么？
rem全称font size of the root element，指**相对于根元素字体大小的单位**。
之前他还有个兄弟：em
- em：相对父元素字体大小的单位
- rem：相对于根元素字体大小的单位

#### 为什么？
理由的话我们之前也讲过了，rem是针对于web app推出的新属性，同样的适用于web page。如果按照之前的固定宽度做法太死板，如果设置响应式布局又难以维护还麻烦，所以干脆咱们用rem，有个统一参照还好收拾，利索！

#### 怎样做
首先是设置根元素，也就是html的字体大小
比如我们设置为10px

```
html{
	font-size:10px;
}
```
然后就可以灵活的控制啦

```
.btn {
    width: 6rem;
    height: 3rem;
    line-height: 3rem;
    font-size: 1.2rem;
    display: inline-block;
    background: #06c;
    color: #fff;
    border-radius: .5rem;
    text-decoration: none;
    text-align: center;    
}
```
最终.btn的宽度为60px

敲黑板！！！！！说结论啦！！！！！
**最终字体的px = 倍数 * 根节点px**
也就是说，上边的60px宽度是6*10px计算得来的！、

##### 不同分辨率下的font-size又是怎么搞？

```
html{
	//100vw是设备的宽度，除以7.5可以让1rem的大小在iPhone6下等于100px。
    font-size: calc(100vw/7.5);
}
```
全局px替换为rem
替换设计稿中的px为rem，即font-size: 12px，需要写成font-size: .12rem。注意rem=px/100，其他宽度、高度、间距等同理。由于rem是相对于根节点元素大小的单位，遂当设备宽度改变时，采用rem布局的大小均会跟随相同比例变化，从而实现整体缩放。

低版本浏览器的话还需要手动适配哦
```
document.documentElement.style.fontSize = window.innerWidth/7.5 + 'px'
```

