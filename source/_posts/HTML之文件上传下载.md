---
title: HTML5之文件上传下载
categories: HTML5
tags:
 - FileList
 - file
 - Blob
 - FileReader
---

h5 所提供的文件 API 在前端领域有着丰富的应用和广泛的兼容，甚至包括移动端。所以我们今天来讲一讲 H5 的文件 API

<!--more-->

### FileList 对象和 file 对象

HTML 中的`input[type="file"]`标签有个`multiple`属性，允许用户选择多个文件。`FileList`对象就是表示用户选择的文件列表。此列表中的每一个文件，就是一个`file`对象。

- `file`对象的属性
  - `name`：文件名，不包含啊路径
  - `type`：文件类型。
  - `size`：文件大小。
  - `lastModified`：文件最后修改时间

```html
<input type="file" id="files" multiple>
<script>
    var elem = document.getElementById('files');
    elem.onchange = function (event) {
        var files = event.target.files;
        for (var i = 0; i < files.length; i++) {
            // 文件类型为 image 并且文件大小小于 200kb
            if(files[i].type.indexOf('image/') !== -1 && files[i].size < 204800){
                console.log(files[i].name);
            }
        }
    }
</script>
```

`input`中有个`accept`属性，可以用来规定能够上传的文件类型。

我们可以通过设置`accept`属性来限制只允许上传图像格式

```html
<input type="file" accept="image/*">
<!-- 或 -->
<input type="file" accept="image/gif,image/jpeg,image/jpg,image/png">
```

### Blob对象

`Blob`对象相当于一个容器，可以用来存放二进制数据。其`size`属性表示字节长度，`type`属性表示`MIME`类型。

我们通过`Blob`构造函数来创建`Blob`对象

```javascript
// 第一个参数是用来存放ArrayBuffer、ArrayBufferView、Blob和字符串的一个数组
// 第二个参数则规定了其MIME类型
var blob = new Blob(['hello'], {type:"text/plain"})
```

#### 下载文件

`Blob`对象可以通过`window.URL`对象生成一个网络地址，结合`a`标签的`download`属性来下载文件

```javascript
// 将一个 canvas 下载为一个图片文件
var canvas = document.getElementById('canvas');
canvas.toBlob(function(blob){
    // 使用 createObjectURL 生成地址，格式为 blob:null/fd95b806-db11-4f98-b2ce-5eb16b38ba36
    var url = URL.createObjectURL(blob);
    var a = document.createElement('a');
    a.download = 'canvas';
    a.href = url;
    // 模拟a标签点击进行下载
    a.click();
    // 下载后告诉浏览器不再需要保持这个文件的引用了
    URL.revokeObjectURL(url);
});
```

当然也可以将字符串保存为一个文本文件

### FileReader对象

`FileReader`对象主要用于将文件读入内存，并读取文件中的数据

````javascript
var reader = new FileReader()
````

该对象有如下方法

- `abort`：中断读取操作
- `readAsArrayBuffer`：读取文件内容到`ArrayBuffer`对象中
- `readAsBinaryString`：将文件读取为二进制数据
- `readAsDataURL`：将文件读取为`data:URL`格式的字符串
- `readAsText`：将文件读取为文本

#### 上传图片预览

常见的就是在客户端上传图片后通过`readAsDataURL()`来显示图片

```html
<input type="file" id="files" accept="image/jpeg,image/jpg,image/png">
<img src="blank.gif" id="preview">
<script>
    var elem = document.getElementById('files'),
        img = document.getElementById('preview');
    elem.onchange = function () {
        var files = elem.files,
            reader = new FileReader();
        if(files && files[0]){
            reader.onload = function (ev) {
                img.src = ev.target.result;
            }
            reader.readAsDataURL(files[0]);
        }
    }
</script>
```

但在一些手机上竖着拍照上传照片时有bug。会发现照片倒了。（手机拍出的照片默认带`Orientation`参数，代表方向。正常为1，可以通过`exif.js`获取）[移动端图片上传旋转、压缩的解决方案](https://github.com/lin-xin/blog/issues/18)

#### 数据备份与恢复

`FileReader`对象的`readAsText()`可以读取文件的文本，结合`Blob`对象下载文件的功能，那就可以实现将数据熬出文件备份到本地。当数据要恢复时，通过`input`把备份文件上传。使用`readAsText()`读取文本，恢复数据。

代码跟上面功能类似。

### Base64 编码

在 H5 中新增了`atob(解码)`和`btoa(编码)`方法来支持`Base64`编码。

```javascript
var a = "https://lin-xin.github.io";
var b = btoa(a);
var c = atob(b);

console.log(a);     // https://lin-xin.github.io
console.log(b);     // aHR0cHM6Ly9saW4teGluLmdpdGh1Yi5pbw==
console.log(c);     // https://lin-xin.github.io
```

`btoa`方法对字符串 a 进行编码，不会改变`a`的值，返回一个编码后的值。
atob 方法对编码后的字符串进行解码。

但是参数中带中文，已经超出了8位ASCII编码的字符范围，浏览器就会报错。所以需要先对中文进行`encodeURIComponent`编码处理。

```javascript
var a = "哈喽 世界";
var b = btoa(encodeURIComponent(a));
var c = decodeURIComponent(atob(b));

console.log(b);     // JUU1JTkzJTg4JUU1JTk2JUJEJTIwJUU0JUI4JTk2JUU3JTk1JThD
console.log(c);     // 哈喽 世界
```

