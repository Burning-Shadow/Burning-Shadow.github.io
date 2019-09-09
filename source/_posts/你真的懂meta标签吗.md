---
title: 你真的懂meta标签吗
categories: JavaScript
tags: 
 - meta标签
 - content-security-policy
 - x-dns-prefetch-control
 - refresh
 - default-style
---

作为一个二五仔之前其实一直对这个属性采取视而不见态度的。直到碰到了一次手机端由于适配问题不强需要采取 PC 端布局的问题才逐渐开始了解。后来看了一些资料逐渐才对这东西有所了解。但是自己才疏学浅，所以大都是抄人家的，结尾会附上参考文章的链接，方便大家拜读原著。

<!--more-->

### 用处

meta常用于定义页面的说明，关键字，最后修改日期，和其它的元数据。这些元数据将服务于浏览器（如何布局或重载页面），搜索引擎和其它网络服务。

### http-equiv

`http-equiv`是 meta 标签中非常重要的一环，其内容值均为特定的 HTTP 头。该属性与服务器和浏览器进行交互，让网站内容现实的准确和及时

- `content-Type`（设定网页字符集）

  - ```
    <meta http-equiv="content-Type" content="text/html;charset=utf-8">	// 旧HTML
    <meta charset="utf-8">		// H5
    ```

- `X-UA-Compatible`：（浏览器采取何种版本渲染当前页面）

  - ```
     <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"/> //指定IE和Chrome使用最新版本渲染当前页面
    ```

- `Set-Cookie`：`cookie`设定

  - ```
    <meta http-equiv="Set-Cookie" content="name, date"> //格式
    
    <meta http-equiv="Set-Cookie" content="User=Lxxyx; path=/; expires=Sunday, 10-Jan-16 10:00:00 GMT"> //具体范例
    ```

- `content-security-policy`：通过设置此属性来声明哪些动态资源允许被加载以减少 XSS 攻击。

- `cache-control`、`Pragma`、`Expires`：这三个属性与 HTTP 头有相同的属性。加上相应的属性能够让浏览器缓存相应的 html 内容。

  - 某些网站上我们可以看到以下 meta 标签内容

  - ```html
    <meta http-equiv="cache-control" content="max-age=180" />
    <meta http-equiv="cache-control" content="no-cache" />
    <meta http-equiv="expires" content="0" />
    <meta http-equiv="expires" content="Tue, 01 Jan 1980 1:00:00 GMT" />
    <meta http-equiv="pragma" content="no-cache" />
    ```

  - **但由于 H5 规范中`http-equiv`中的属性并不包括这三个，故缓存控制的任务最后还是落在了 HTTP headers 上**

- `x-dns-prefetch-control`：我们可以通过此属性来控制meta标签来提前进行 DNS 解析，以此来提升网站性能。

  - HTML 页面中`<a>`会自动启用 DNS 提前解析，但在 HTTPS 上却失效了。此时通过设置`<meta http-equiv="x-dns-prefetch-control" content="on" />`来让`<a>`得以在 HTTPS 环境下进行DNS预解析

- `refresh`（自动刷新并指向某页面）：此属性可以用作页面的跳转，作用同`window.location.href`

  - ```html
    //当前页面每一秒后刷新一下
    <meta http-equiv="refresh" content="1">
    //当前页面一秒后跳转到首页
    <meta http-equiv="refresh" content="0;url='/'">
    /当前页面一秒后跳转到百度
    <meta http-equiv="refresh" content="0;url='https://www.baidu.com'">
    ```

  - 不过`FireFox`需要手动启用`autorefreh`，同时其相对于 JS 执行，跳转需要等待事件过长。

- `default-style`：指定了页面上使用的首选样式表。`content`属性必须包含`<link>`元素的标题，`href`属性了解到 CSS 样式表或包含 CSS 样式表的`<style>`元素的标题

### name

- `keywords`：网页关键字：告知搜索引擎你网页的关键字

```html
<meta name="keywords" content="Lxxyx,博客，文科生，前端">
```

- `description（网站内容的描述）`：用于告知搜索引擎你网站的主要内容

- `viewport`（移动端的窗口）：我们下面会讲到

- `robots`（定义搜索引擎爬虫的索引方式）：告诉爬虫哪些需要索引，哪些不需要

  - **这个标签会对搜索引擎优化（SEO）起作用**。它可以防止对拷贝内容的冗余抓取

  - <meta name="robots" content="none">

    - `none`：搜索引擎将忽略此网页，等价于`noindex`，`nofollow`。
    - `noindex`：搜索引擎不索引此网页。
    - `nofollow`：搜索引擎不继续通过此网页的链接索引搜索其它的网页。
    - `all`：搜索引擎将索引此网页与继续通过此网页的链接索引，等价于`index`，`follow`。
    - `index：`搜索引擎索引此网页。
    - `follow：`搜索引擎继续通过此网页的链接索引搜索其它的网页。

### viewport

这东西就是咱们最常用的页面布局啦。我们使用 boorstrap 框架的时候就需要对其进行设置。

```html
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no,minimal-ui">
```

- `width`：`viewport`的宽度
- `height`：`viewport`的高度（范围从223到10000）
- `user-scalable`：`[yes | no]`是否允许缩放
- `initial-scale`：【数值】初始化比例（范围从 0 ~ 10）
- `minimum-scale`：【数值】允许缩放的最小比例
- `maximum-scale`：【数值】允许缩放的最大比例
- `minimal-ui`：IOS7.1的新属性，最小化浏览器UI

### 参考文章

<https://segmentfault.com/a/1190000004279791#articleHeader2>

<https://juejin.im/post/5c288546e51d451a6b51554a#heading-1>

<https://www.notion.so/meta-8cd558ba52d843058292b5beb485ee1f>