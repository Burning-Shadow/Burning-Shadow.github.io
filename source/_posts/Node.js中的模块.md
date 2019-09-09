---
title: Node.js中的模块
categories: Node.js
tags:
 - require
 - exports
 - module
 - 初始化
 - 主模块
---

编写稍大的程序时一般都会将代码模块化。在NodeJS中，一般将代码合理拆分到不同的JS文件中，每一个文件就是一个模块，而文件路径就是模块名。

<!--more-->

### require

`require`函数用于在当前模块中**加载和使用别的模块**。

传入一个模块名，返回一个模块到处的对象。此种方法甚至可以加载和使用一个 JSON 文件。

### exports

`exports`对象是当前模块的导出对象，用于**导出模块公有方法和属性**。

别的模块通过`require`函数使用当前模块时得到的就是当前模块的`exports`对象。以下例子中导出了一个公有方法。

```javascript
exports.hello = function () {
    console.log('Hello World!');
};
```

### module

通过`module`对象可以访问到当前模块的一些相关信息，但最多的用途是**替换当前模块的导出对象**。

例如模块导出对象默认是一个普通对象，如果想改成一个函数的话，可以使用以下方式。

```javascript
module.exports = function(){
    console.log('Hello World')
}
```

### 模块初始化

一个模块中的JS代码仅在模块第一次被使用时执行一次，并在执行过程中初始化模块的导出对象。

之后，缓存起来的导出对象被重复利用。

### 模块路径解析规则

我们已经知道，`require`函数支持斜杠（`/`）或盘符（`C:`）开头的绝对路径，也支持`./`开头的相对路径。

但这两种路径在模块之间建立了强耦合关系，一旦某个模块文件的存放位置需要变更，使用该模块的其它模块的代码也需要跟着调整，变得牵一发动全身。

因此，`require`函数支持第三种形式的路径，写法类似于`foo/bar`，并依次按照以下规则解析路径，直到找到模块位置。

#### 内置模块

如果传递给`require`函数的是NodeJS内置模块名称，不做路径解析，直接返回内部模块的导出对象，例如`require('fs')`。

#### node_modules目录

NodeJS定义了一个特殊的`node_modules`目录用于存放模块。例如某个模块的绝对路径是`/home/user/hello.js`，在该模块中使用`require('foo/bar')`方式加载模块时，则NodeJS依次尝试使用以下路径。

```javascript
/home/user/node_modules/foo/bar
/home/node_modules/foo/bar
/node_modules/foo/bar
```

#### NODE_PATH环境变量

NodeJS允许通过NODE_PATH环境变量来指定额外的模块搜索路径。NODE_PATH环境变量中包含一到多个目录路径，路径之间在Linux下使用`:`分隔，在Windows下使用`;`分隔。例如定义了以下NODE_PATH环境变量：

```javascript
NODE_PATH=/home/user/lib:/home/lib
```

当使用`require('foo/bar')`的方式加载模块时，则NodeJS依次尝试以下路径。

```javascript
/home/user/lib/foo/bar
/home/lib/foo/bar
```

### 包(package)

复杂的模块往往有多个子模块组成。为了方便管理和使用，我们可以将由多个子模块组成的大模块称为包，并将所有子模块放在同一个目录来里。

在组成一个包的所有子模块中，需要有一个入口模块，入口模块的导出对象被作为包的导出对象。例如有以下目录结构。

```
- /home/user/lib/
    - cat/
        head.js
        body.js
        main.js
```

其中`cat`目录定义了一个包，其中包含了3个子模块。`main.js`作为入口模块，其内容如下：

```
var head = require('./head');
var body = require('./body');

exports.create = function (name) {
    return {
        name: name,
        head: head.create(),
        body: body.create()
    };
};
```

在其它模块里使用包的时候，需要加载包的入口模块。接着上例，使用`require('/home/user/lib/cat/main')`能达到目的，但是入口模块名称出现在路径里看上去不是个好主意。因此我们需要做点额外的工作，让包使用起来更像是单个模块。

#### index.js

当模块的文件名是`index.js`，加载模块时可以使用模块所在目录的路径代替模块文件路径，因此接着上例，以下两条语句等价。

```
var cat = require('/home/user/lib/cat');
var cat = require('/home/user/lib/cat/index');
```

这样处理后，就只需要把包目录路径传递给`require`函数，感觉上整个目录被当作单个模块使用，更有整体感。

#### package.json

如果想自定义入口模块的文件名和存放位置，就需要在包目录下包含一个`package.json`文件，并在其中指定入口模块的路径。上例中的`cat`模块可以重构如下。

```
- /home/user/lib/
    - cat/
        + doc/
        - lib/
            head.js
            body.js
            main.js
        + tests/
        package.json
```

其中`package.json`内容如下。

```
{
    "name": "cat",
    "main": "./lib/main.js"
}
```

如此一来，就同样可以使用`require('/home/user/lib/cat')`的方式加载模块。NodeJS会根据包目录下的`package.json`找到入口模块所在位置。

