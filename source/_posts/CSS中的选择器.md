---
title: CSS中的选择器
date: 2019-04-05 11:30:00
categories: CSS
tags:
 - 基本选择器
 - 属性选择器
 - 伪类选择器
 - 伪元素选择器
 - 多级选择器
 - 选择器优先级
---

本文介绍了 CSS 中的选择器

<!--more-->

### 基本选择器

- `*`：通配符选择器
- `#`：id选择器
- `.`：类选择器
- `element`：元素选择器

### 属性选择器

- `[attribute]`：匹配所有带`attribute`属性的元素
- `[attribute="x"]`：匹配所有`attribute`属性为x的元素
- `[attribute~="x"]`：匹配所有`attribute`属性具有多个空格分割的值，其中一个值等于x的元素
- `[attribute|="x"]`：匹配所有`attribute`属性具有连字符`-`分隔的值，其中一个值以 x 开头的元素



- `[attribute^="x"]`：匹配所有`attribute`属性以x开头的值
- `[attribute$="x"]`：匹配所有`attribute`属性以x结尾的值
- `[attribute*="x"]`：匹配属性`attribute`属性的值包含x的元素

### 伪类选择器

- `E:first-child`: 匹配元素 E 当它是其父元素的第一个子元素
- `E:link`: 匹配未被访问(未点击或跳转)的链接
- `E:visited`: 匹配已访问过的链接
- `E:active`: 匹配鼠标按下还未抬起的元素`E:hover`: 匹配鼠标悬停其上的元素
- `E:focus`: 匹配获取当前焦点的元素
- `E:lang(x)`: 匹配 `lang` 属性等于 x 的元素



- `E:target`: URL 后跟锚点#，指向文档内某个具体的元素，这个被链接的元素就是目标元素，`E:target`选 择器用于选取当前活动的目标元素 当我们点击列表 tab1 时，因为其锚点链接的元素就是 id 为 tab1 的元素，所以此时活动的目标元素就是 id 为 tab1 的 div，通过 div:target 就可以获取此目标元素。
- `:not(selector)`: 匹配与 selector 选择器描述不相符的元素
- `E:enabled`: 匹配表单中激活的元素
- `E:disabled`: 匹配表单中禁用的元素
- `E:checked`: 匹配表单中被选中的 radio(单选框)或 checkbox(复选框)

#### CSS3结构性伪类

- `:root`: 匹配根元素，对应 HTML 文档就是 html 元素
- `E:nth-child(n)`: 匹配元素 E 当它是其父元素的第 n(从 1 开始)个子元素 列表 tab1 是其父元素 ul 的第一个元素，因此可以匹配到列表 tab1
- `E:nth-last-child(n)`: 匹配元素 E 当它是其父元素的倒数第 n(从 1 开始)个子元素 列表 tab2 是其父元素 ul 的倒数第一个子元素，因此可以匹配到列表 tab2
- `E:last-child`: 匹配元素 E 当它是其父元素的倒数第 1 个子元素
- `E:only-child`: 匹配元素 E 当它是其父元素的唯一一个子元素
- `E:nth-of-type(n)`: 匹配元素 E 当它是其父元素的第 n(从 1 开始)个出现的与 E 类型相同的子元素
- `E:nth-last-of-type(n)`: 匹配元素 E 当它是其父元素的倒数第 n(从 1 开始)个出现的与 E 类型相同的子元素
- `E:first-of-type`: 匹配元素 E 当它是其父元素的第 1 个出现的与 E 类型相同的元素子元素(可能有多个)
- `E:last-of-type`: 匹配元素 E 当它是其父元素的倒数第 1 个出现的与 E 类型相同的元素子元素(可能有多个)
- `E:only-of-type`: 匹配元素 E 当它是其父元素下唯一一个 E 类型的元素
- `E:empty`: 匹配元素 E 当没有子元素或内容时

### 伪元素选择器

- `::first-line`: 匹配元素的第一行
- `::first-letter`: 匹配元素的第一个字母
- `::before`: 在元素前通过 content 属性插入内容
- `::after`: 在元素后通过 content 属性插入内容
- `::selection`: 匹配鼠标框选的元素

### 多级选择器

- `E, F`: 多元素选择器，同时匹配 E 元素和 F 元素
- `E > F`: 子元素选择器，匹配 E 元素的子元素 F
- `E F`: 后代元素选择器，匹配 E 元素的后代元素 F
- `E + F`: 相邻元素选择器，匹配所有紧随 E 元素之后的 F 元素 匹配 div3
- `E ~ F`: 同级元素选择器，匹配所有 E 元素之后的同级元素 F 匹配 div3，div4

### 优先级

`!important > 行内样式 > ID > 类、伪类、属性 > 元素、伪元素 > 继承 > 通配符`