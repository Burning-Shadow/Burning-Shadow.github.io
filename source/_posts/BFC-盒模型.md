---
title: BFC 盒模型
categories: CSS
tags:
 - BFC
---

### 引言

盒模型又成为框模型（`Box Model`），包含了元素内容（`Content`）、内边距（`padding`）、边框（`border`）、外边距（`margin`）几个要素。

盒模型分为两种—— 标准盒模型 & IE模型

> 标准模型：即 W3C 盒模型，`width = content`
>
> IE 模型：`width = conten + padding`

<!--more-->

我们可以通过 CSS3 新增的属性`box-sizing`将盒模型设置为标准模型（`content-box`）和 IE 模型（`border-box`）

### 外边距重叠

![](https://pic.superbed.cn/item/5c99dde53a213b041727474c)

当两个垂直外边距相遇时，他们将形成一个外边距，合并后的外边距高度等于两个发生合并的外边距的高度中的较大者。**注意**：只有**普通文档流**中**块框的垂直外边距**才会发生外边距合并，行内框、浮动框或绝对定位之间的外边距不会合并。

### 咋整？

这就牵扯出咱们的 BFC 了。

> **BFC(Block Formatting Context)**：块级格式化上下文。

​	BFC 决定了元素如何对其内容进行定位，以及与其他元素的关系和相互作用。当设计到可视化布局的时候，BFC提供了一个环境，HTML元素在这个环境中按照一定的规则进行布局。一个环境中的元素不会影响到其他环境中的布局。

#### **BFC的原理（渲染规则）**

1. BFC元素垂直方向的边距会发生重叠。属于不同BFC外边距不会发生重叠
2. BFC的区域不会与浮动元素的布局重叠。
3. BFC元素是一个独立的容器，外面的元素不会影响里面的元素。里面的元素也不会影响外面的元素。
4. 计算BFC高度的时候，浮动元素也会参与计算(清除浮动)

#### 如何创建 BFC

1. `overflow`不为`visible`;
2. `float`的值不为`none`；
3. `position`的值不为`static`或`relative`；
4. `display`属性为`inline-blocks`,`table,table-cell`,`table-caption`,`flex,inline-flex`;

#### 碰到浮动怎么办

- 子元素浮动后自动脱离文档流。此时父元素就不会计算它的高度。而此时在父元素创建 BFC 就可以让浮动元素也参加高度计算



