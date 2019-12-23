---
title: react-dom-router初体验
date: 2019-12-23 22:00:26
categories: React
tags: 
 - react-router-dom
---

​		不出意外今天又很懒惰，不想写代码也不想开新坑也不想补老坑，所以就把最近项目中用到的 `react-router-dom` 写一篇博客吧~

​		作为 `react-router4` 的船新版本 `react-router-dom` 颠覆了传统的路由格式，将原本的格式化路由变成了组件化，无疑算是一大革新。其优劣暂且避开不谈，光是创新就让人们耳目一新了。网络对此褒贬不一，但本着有新用新的原则在我新开的坑里还是毅然决然的使用了 `react-router4` 。那么现在试着给大伙讲明白吧~

​		在正式开始之前我们将 [模拟训练的教程网站]( https://reacttraining.com/react-router/web/example/basic ) 放在最前边，方便大家根据例子进行学习。当然，跟着我也行。

<!--more-->

> `react-router-dom` 其最大的革新点在于将原本的 `router.js` 模式的路由映射文件改为组件化形式，我们只需要关注跳转（`Link`）和组件（`Route`），无需关心其他操作。

​		