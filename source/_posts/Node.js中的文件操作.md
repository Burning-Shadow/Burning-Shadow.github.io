---
title: Node.js中的文件操作
date: 2019-04-28 20:55:00
categories: Node.js
tags:
 - fs
 - buffer
 - stream
 - file system
 - path
---

`fs`作为Node.js最强大的模块，小到文件查找，大至代码编译，均需经过它之手。关键人家还是自带模块，无需下载安装，直接引用即可。所以我们今天来聊一聊其内置的文件模块

<!--more-->

### 文件拷贝

Node提供了基本的文件操作API，但是如文件拷贝这样的高级功能并未提供。所以我们来试着实现一下

#### 小文件拷贝

```javascript
// 使用 fs.readFileSync 从源路径读取文件内容，并使用 fs.writeFileSync 将文件内容写入目标路径
var fs = require('fs');

function copy(src, dst) {
    fs.writeFileSync(dst, fs.readFileSync(src));
}

function main(argv) {
    copy(argv[0], argv[1]);
}

main(process.argv.slice(2));
```

> `process`是一个全局变量，可通过`process.argv`获取命令行参数。
>
> 由于`argv[0]`固定等于NodeJS执行程序的绝对路径，`argv[1]`固定等于主模块的绝对路径，因此第一个命令行参数从`argv[2]`开始。

#### 大文件拷贝

上边的程序对于小文件来说没有问题，但是大文件显然无法通过将所有文件内容读取到磁盘中后再一次性写入。

所以我们只能读一点写一点，直至完成拷贝

```javascript
var fs = require('fs');

function copy(src, dst) {
    fs.createReadStream(src).pipe(fs.createWriteStream(dst));
}

function main(argv) {
    copy(argv[0], argv[1]);
}

main(process.argv.slice(2));
```

以上程序使用`fs.createReadStream`创建了一个源文件的只读数据流，并使用`fs.createWriteStream`创建了一个目标文件的只写数据流，并且用`pipe`方法把两个数据流连接了起来。

连接起来后发生的事情，说得抽象点的话，水顺着水管从一个桶流到了另一个桶。 

### Buffer(数据块)

`Buffer`相当于是字符串形式的二进制数据类型。

为什么这么说呢？因为 JS 语言自身只有字符串数据类型，这个”二进制“是咱们模拟出来的，用以提供对二进制数据的操作。

#### toString

我们可以通过`toString`设置`Buffer`字符串的编码格式，同样，我们也可以转回去，将字符串变为制定编码下的二进制数据

```javascript
var bin = new Buffer([0x64, 0x65, 0x6c, 0x6f])
var str = bin,toString('utf-8')		// 'hello'
var a = new Buffer('hello', 'utf-8')	// <Buffer 68 65 6c 6c 6f>
```

#### slice

`.slice`方法并非返回了一个新的`Buffer`，而是像返回了指向原`Buffer`中间的 某个位置的指针

```javascript
[ 0x68, 0x65, 0x6c, 0x6c, 0x6f ]
    ^           ^
    |           |
   bin     bin.slice(2)
```

因此`.slice`方法返回的`Buffer`的修改会作用域原`BUffer`

```javascript
var bin = new Buffer([ 0x68, 0x65, 0x6c, 0x6c, 0x6f ]);
var sub = bin.slice(2);

sub[0] = 0x65;
console.log(bin); // => <Buffer 68 65 65 6c 6f>
```

因此，需要拷贝`Buffer`时必须创建一个新的`Buffer`，并通过`.copy`方法将原`Buffer`中的数据复制过去。这就类似于申请一块新的内存并将内存中的数据复制过去

```javascript
var bin = new Buffer([ 0x68, 0x65, 0x6c, 0x6c, 0x6f ]);
var dup = new Buffer(bin.length);

bin.copy(dup);
dup[0] = 0x48;
console.log(bin); // => <Buffer 68 65 6c 6c 6f>
console.log(dup); // => <Buffer 48 65 65 6c 6f>
```

总之，`Buffer`将JS的数据处理能力从字符串扩展到了任意二进制数据。

### Stream(数据流)

当我们需要一边读取一边处理数据时，就需要用到数据流。NodeJS中通过各种`Stream`来提供对数据流的操作

我们在开始就介绍了大文件拷贝的例子。我们可以为数据来源创建一个只读数据流

```javascript
var rs = fs.createReadStream(pathname);

rs.on('data', function (chunk) {
    doSomething(chunk);
});

rs.on('end', function () {
    cleanUp();
});
```

> `Stream`基于事件机制工作，所有`Stream`的实例都继承于NodeJS提供的`EventEmitter`

上面代码中的`data`事件会源源不断的被触发，不管`doSomething`函数是否处理的过来，代码可以继续做如下改造，以解决这个问题。

```javascript
var rs = fs.createReadStream(src);

rs.on('data', function (chunk) {
    rs.pause();
    doSomething(chunk, function () {
        rs.resume();
    });
});

rs.on('end', function () {
    cleanUp();
});
```

我们为`doSomething`函数加上了回调，如此就可以在处理数据前暂停数据读取，在处理数据后恢复数据读取

此外，还可以为数据目标创建一个只写数据流

```javascript
var rs = fs.createReadStream(src);
var ws = fs.createWriteStream(dst);

rs.on('data', function (chunk) {
    ws.write(chunk);
});

rs.on('end', function () {
    ws.end();
});
```

我们将`doSomething`换成了往只写数据流中写入数据后，以上代码看起来就更像一个文件拷贝程序了。但是还是存在上述提到的问题：若写入速度跟不上读取速度的话，只写数据流内部的缓存会爆仓。

我们可以根据`.write`方法的返回值来判断传入的数据是已写入目标，还是临时放在了缓存中。并根据`drain`事件来判断什么时候只写数据流已将缓存中的数据写入目标，可以传入下一个待写数据了

```javascript
var rs = fs.createReadStream(src);
var ws = fs.createWriteStream(dst);

rs.on('data', function (chunk) {
    if (ws.write(chunk) === false) {
        rs.pause();
    }
});

rs.on('end', function () {
    ws.end();
});

ws.on('drain', function () {
    rs.resume();
});
```

后续，NodeJS直接提供了`.pipe`方法来做这些事情，内部原理和以上代码类似，并包括了防爆仓控制。

### File System(文件系统)

`fs`模块提供的API通常可以分为以下三类

- 文件属性读写：`fs.stat`、`fs.chmod`、`fs.chown`等等
- 文件内容读写：`fs.readFile`、`fs.readdir`、`fs.writeFile`、`fs.mkdir`
- 底层文件操作：`fs.open`、`fs.read`、`fs.wite`、`fs.close`

NodeJS最精华的异步IO模型在`fs`模块里有着充分的体现，例如上边提到的这些API都通过回调函数传递结果。

```javascript
fs.readFile(pathname, function (err, data) {
    if (err) {
        // Deal with error.
    } else {
        // Deal with data.
    }
});
```

如上边代码所示，基本上所有`fs`模块API的回调参数都有两个。第一个参数在有错误发生时等于异常对象，第二个参数始终用于返回API方法执行结果。

此外，`fs`模块的所有异步API都有对应的同步版本，用于无法使用异步操作时，或者同步操作更方便时的情况。同步API除了方法名的末尾多了一个`Sync`之外，异常对象与执行结果的传递方式也有相应变化。同样以`fs.readFileSync`为例：

```javascript
try {
    var data = fs.readFileSync(pathname);
    // Deal with data.
} catch (err) {
    // Deal with error.
}
```

`fs`模块提供的API很多，这里不一一介绍，需要时请自行查阅官方文档。

### Path(路径)

- `path.normalize`

  - 将传入的路径转换为标准路径，具体讲的话，出去解析路径中的`.`与`..`外，还能去掉许多多余的斜杠。如果有程序需要使用路径作为某些数据的索引，但又允许用户随意输入路径时，就需要使用该方法保证路径的唯一性

  - ```javascript
    var cache = {};
    
    function store(key, value) {
        cache[path.normalize(key)] = value;
    }
    
    store('foo/bar', 1);
    store('foo//baz//../bar', 2);
    console.log(cache);  // => { "foo/bar": 2 }
    ```


### 其他API

- `stat()`查看文件的详细信息

  - ```javascript
    const fs = require('fs');
    
    fs.stat('fileUrl', (err, data) => {
        if (err) {
            throw err;//这里可以判断文件是否存在
        }
        
        console.log(data);
    });
    ```

- `rename()`：更改文件名

  - ```javascript
    fs.rename('./text.txt','hahah.ttx');
    ```

- `unlink`：删除文件

  - ```javascript
    fs.unlink('fileName', err => err);
    ```

- `readdir()`：读取文件夹

- `mkdir()`：创建文件夹

- `rmdir()`：删除文件夹

- `watch()`：监视文件或目录变化

  - ```javascript
    fs.watch('fileUrl', {
        recursive:true //是否监视子文件夹
    }, (eventType, fileName) => {
        console.log(eventType, fileName);
    })
    ```

- `readStream()`：读取流

  - ```javascript
    const rs = fs.createReadStream('urlPath');
    
    rs.pipe(process.stdout);//导出文件到控制台
    ```

- `writeStream()`：写入流

- `pipe()`：管道，导通流文件

  - ```javascript
    const ws  = fscreateWriteStream('urlPath');
    
    ws.write('some content');
    
    ws.end();
    
    ws.on('finish',()=>{
       console.log('done!!!'); 
    });
    ```

