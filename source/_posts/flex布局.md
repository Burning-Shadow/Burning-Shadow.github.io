---
title: Flex布局
categories: CSS
---

#### 简介
- flex是Flexible Box的缩写，即“弹性布局”
- **设置display:flex;属性后，子元素的float,clear和vertical-align属性将不再起作用**
- 此属性既适用于块级元素也适用于行内元素

<!--more-->

#### 概念

display属性为flex的元素称为**容器**，其所有子元素则为**容器成员**，称为flex item（flex项目）。

容器默认有x,y两条轴：水平的称为主轴（main axis），竖直的称为交叉轴（cross axis），其余详细信息我们用一幅图来演示

![](https://pic.superbed.cn/item/5c94f9f33a213b0417e5bfc1)

- 了解了这些之后我们看一下flex的一些子属性：
    - flex-direction　　容器内项目的排列方向(默认横向排列)　　
    - flex-wrap　　容器内项目换行方式
    - flex-flow　　以上两个属性的简写方式
    - justify-content　　项目在主轴上的对齐方式
    - align-items　　项目在交叉轴上如何对齐
    - align-content　　定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用

接下来我们对这些属性做一个详细的介绍

- flex-direction:
    - row(默认)沿水平主轴由左向右排列
    - row-reverse沿水平主轴由右向左排列
    - column沿垂直主轴由上到下
    - column-reverse沿垂直主轴由下到上
- flex-warp：
    - nowrap（默认）：不换行
    - wrap：换行，第一行在上方
    - wrap-reverse：换行，第一行在下方
- flex-flow:
    - flex-flow属性是flex-direction属性和flex-wrap属性的简写形式，默认值为row nowrap
    - flex-flow:<flex-direction> || <flex-wrap>
- justify-content：
    - 话不多说上图
      ![image](https://img-blog.csdn.net/20180617202022711?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMzUzNjYy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
      ![image](https://pic.superbed.cn/item/5c94fa493a213b0417e5c2f7)

- align-items：
    - flex-start：交叉轴的起点对齐。
    - flex-end：交叉轴的终点对齐。
    - center：交叉轴的中点对齐。
    - baseline: 项目的第一行文字的基线对齐。
    - stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。
      ![image](https://pic.superbed.cn/item/5c94fab73a213b0417e5c6d5)
- align-content:
    - flex-start：与交叉轴的起点对齐。
    - flex-end：与交叉轴的终点对齐。
    - center：与交叉轴的中点对齐。
    - space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
    - space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
    - stretch（默认值）：轴线占满整个交叉轴。
      ![image](https://pic.superbed.cn/item/5c94fad23a213b0417e5c7ce)
#### 项目（子元素）同样也有属性
- order：
    - order属性定义项目的排列顺序。数值越小，排列越靠前，默认为0。
      ![image](https://pic.superbed.cn/item/5c94faed3a213b0417e5c92f)
- flex-grow：
    - flex-grow属性定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大
      ![image](https://pic.superbed.cn/item/5c94fb0c3a213b0417e5ca5d)
- flex-shrink
    - flex-shrink属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。
      ![image](https://pic.superbed.cn/item/5c94fb1e3a213b0417e5cb74)
      如果所有项目的flex-shrink属性都为1，当空间不足时，都将等比例缩小。如果一个项目的flex-shrink属性为0，其他项目都为1，则空间不足时，前者不缩小。

**负值对该属性无效。**
- flex-basis
    - flex-basis属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即项目的本来大小。
    - 它可以设为跟width或height属性一样的值（比如350px），则项目将占据固定空间

```
.item {
  flex-basis: <length> | auto; /* default auto */
}
```
- flex属性
    - flex属性是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。后两个属性可选
    - 该属性有两个快捷值：auto (1 1 auto) 和 none (0 0 auto)。
- 建议优先使用这个属性，而不是单独写三个分离的属性，因为浏览器会推算相关值。
- align-self：
    - **align-self属性允许单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。**