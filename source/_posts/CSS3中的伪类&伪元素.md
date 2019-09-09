---
title: CSS3中的伪类&伪元素
categories: CSS
tags:
 - CSS3
 - 伪类
 - 伪元素 
---

在正式开始之前我们先看一下他们的定义

> 伪元素（Pseudo-elements）
>
> Pseudo-elements create abstractions about the document tree beyond those specified by the document language. For instance, document languages do not offer mechanisms to access the first letter or first line of an element’s content. Pseudo-elements allow authors to refer to this otherwise inaccessible information. Pseudo-elements may also provide authors a way to refer to content that does not exist in the source document.
>
> 核心就是需要创建通常不存在于文档中的元素

> 伪类**(pseudo-classes)**
>
> The pseudo-class concept is introduced to permit selection based on information that lies outside of the document tree or that cannot be expressed using the other simple selectors.
>
> 核心就是用来选择那些**不能够被普通选择器选择**的**文档之外的元素**。比如:hover

**所以，伪类和伪元素都是用来表示文档树以外的“元素”**

**二者的区别关键在于：若没有伪元素（或伪类），是否需要添加元素才能达到目的。若是则是伪元素，反之则是伪类**

<!--more-->

我们看一下他们都包括什么

![](https://pic.superbed.cn/item/5ca626383a213b0417ae4cbd)

![](https://pic.superbed.cn/item/5ca6265e3a213b0417ae4e00)

### 伪元素

所谓的伪元素，顾名思义，假元素。**需要通过添加元素才能达到效果**

当然，这里需要强调的一点是，此处所说的“前”、“后”是指**目标元素里面内容的前后**。所以说，**伪元素是目标元素的子元素**。



我们来看一个栗子：将首字母颜色变为红色

```html
<p><span style={{ color: red }}>I<span/> am snow</p>
```

我们可以通过这种方式达到

然而还可以通过伪元素方式：

```html
p::first-letter {color: red}
<p>I am snow</p>
```

### 伪类

而如果我们需要将一个结构为下图的文档中字的颜色变为红色时

```html
<div>
 <p>I am snow</p>
<div>
```

我们可以通过给元素添加一个类`red-line`来解决

```html
.red-line {
   color: red;
}

<div class='red-line'>
 <p>I am snow</p>
<div>
```

当然，伪类伪类，就是可以达到**类**的效果

```html
div:first-child {
 color: red;
}

<div class='red-line'>
 <p>I am snow</p>
<div>
```

如上就是伪类和伪元素的妙用咯

### 实战

#### 清除浮动

实战中最经典的也就是当初的清除浮动章节咯

```css
.clearfix:after {content:"."; display:block; height:0; visibility:hidden; clear:both; }
.clearfix { *zoom:1; }
```

#### 伪元素实现换行，替代`<br>`

由于`<br>`标签不仅在可维护性方面很差，还污染了结构层的代码。所以我们可以通过运用`after`伪元素来代替换行标签

```css
.inline-element::after{
    content: '\A';
    white-space: pre;
}
```

通过给元素的`after`伪元素添加`content`为`\A`的值。

这里的`\A`是专门代替换行符的`0x000A`。在 CSS 中他可以写作`“\000A”`，或简化为`\A`。这里我们用它来作为`::after`伪元素的内容，相当于在末尾添加了一个换行符的意思

而`white-space: pre`则是为了保留元素后面的空白符和换行符。结合二者就可以实现在行内元素末尾实现换行啦

#### 伪元素完成雪碧图的边距切换

当我们需要将样式中的雪碧图从小背景板切换为更大的背景板的时候，就会存在一个预留边距的问题。一旦开始预留位置不够那么将出现其他标签。而我们又不可能为了一个小小的`icon`去添加新的标签（不符合语义化），所以我们就可以通过伪元素来解决

在背景板（一般是按钮）中设置一个伪元素，将伪元素的狂傲设置为原本`icon`大小，再利用绝对定位将其固定到需要的位置。如此，不论每个雪碧图所在的`icon`边距为多少，都能够完美适应

#### 形变恢复

当我们运用 CSS3 的`transform`属性时我们可以对块级元素进行旋转等倒错。但是上面的文字也会随之扭曲。所以我们可以以一个`div`作为背景板，而文字则是放在另一个层级高于此`div`的`div`中。

通过运用伪类我们可以省去这个不符合语义化的`div`：运用`::before`伪元素，将 CSS3 变换作用于伪元素上，如此一来形变就不会作用于位于`div`上的文字，且并未使用多余的标签。