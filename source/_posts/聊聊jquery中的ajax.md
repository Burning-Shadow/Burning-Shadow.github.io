---
title: 聊聊jquery中的ajax
---

jq的ajax方法封装了原生的阿贾克斯请求，让用户得以更加方便的使用。

<!--more-->

我们来看一下双方的对比

html部分：

```
<form>
   <input type="text" name="username" id="username"><br>
    <input type="password" name="password" id="password"><br>
    <input type="button" id="btn">
</form>
```

- 原生：

```
document.getElementById('btn').addEventListener('click', () => {
    let xhr = new XMLHttpRequest();
    xhr.open('post', 'http://192.168.1.171:80/actas/user/checkUser.action');
    xhr.onreadystatechange = function() {
        if (xhr.status == 200 && xhr.readyState == 4) {
            console.log(xhr.responseText)
        }
    }
    xhr.setRequestHeader('Content-Type', 'application/json');
    xhr.send(JSON.stringify({
        username: document.getElementById('username').value,
        password: document.getElementById('password').value
    }))
})
```

 - jquery：
```
$('#btn').click(function() {
    $.ajax({
    url: 'http://192.168.1.171:80/actas/user/checkUser.action',
    async: true, 	//是否异步
    // timeout:10000,   //超时10s则请求结束
    dataType: 'json',
    type: "POST",
    data: JSON.stringify({
        username: $('#username').val(),
        password: $('#password').val()
    }),
    crossDomain: true, //跨域：true
    contentType: 'application/json',
    // contentType: 'application/x-www-form-urlencoded',
    success: (data) => {
        console.log(data);
    },
    error: (err) => {
        console.log(err)
    },
    complete: () => {
		console.log('complete')
    }
})
```
显而易见，使用ajax实现相较于原生简单得多，也清晰的多

#### 各属性用途

下面我们来具体的讲一下ajax的各个属性，加深大家的理解

- url：**发送请求的地址**
- type：**请求方式，默认为GET**
  - POST
  - GET
  - OPTION
  - PUT（部分浏览器支持）
  - DELETE（部分浏览器支持）
- dataType：**预期服务器返回的数据类型。如果不指定，jQuery 将自动根据 HTTP 包 MIME 信息来智能判断**
  - xml: 返回 XML 文档
  - html: 返回纯文本 HTML 信息；包含的script标签会在插入dom时执行
  - script: 返回纯文本 JavaScript 代码。不会自动缓存结果。除非设置了"cache"参数。
    - 在远程请求时(不在同一个域下)，所有POST请求都将转为GET请求。(因为将使用DOM的script标签来加载)
  - json: 返回 JSON 数据
  - jsonp: JSONP 格式。使用 JSONP 形式调用函数时，如 "myurl?callback=?" jQuery 将自动替换 ? 为正确的函数名，以执行回调函数。
  - text: 返回纯文本字符串
- data：**要发送的数据**
  - 这里注意：若是想要发送json格式的数据，那我们需要将data做一些小的修饰：
  - `data: JSON.stringify({
           username: $('#username').val(),
           password: $('#password').val()
       })`。当然，我们的dataType属性同时也要设置为“JSON”，而这配合食用更佳
- success：请求成功后执行的函数
- error：请求致败后执行的函数
- complete：请求结束后执行的函数（无论成功失败）

诺，这些是常用的一些属性值。但是我们想要深入理解还需要一些更高级的属性：

- contentType：**发送至服务器时内容的编码类型**
  - application/x-www-form-urlencoded：默认值，这种格式的特点就是，name/value 成为一组，每组之间用 & 联接，而 name与value 则是使用 = 连接
    - 不过如果设计页面的人比较缺德，非得给你嵌套一下子，（比如要发送的数据为`{
      data: {
      a: [{
      x: 2
      }]
      }
      }`），这时候你就比较难受了。然后就引出了我们下面要介绍另一个值：
  - application/json：将值设置为它的时候，我们就可以发送json格式的数据了！不过前面也提到了，我们还需要将另外1个属性处理一下：
    - 需要将data后边跟的东西加上JSON.stringfy()
    - 举个栗子：`data: JSON.stringify({a: [{b:1, a:1}]})`

#### Important！！！！
- headers：**用于设置头部信息！！！！重中之重**

……未完待更

