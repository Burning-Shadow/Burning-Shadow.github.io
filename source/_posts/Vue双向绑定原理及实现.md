---
title: Vue双向绑定原理及实现
categories: Vue
tags: 
 - 双向绑定
 - 数据劫持
 - 订阅者-发布者模式


---

`Vue`实现双向绑定的原理就是通过数据劫持结合发布者-订阅者模式的方式来实现的。

但是数据劫持是什么？咱们先来看看`Object.defineProperty()`

<!--more-->

开始之前先用原作者的代码和演示图给大家展示一下效果

![](https://0d077ef9e74d8.cdn.sohucs.com/rmcZAcs_jpg)

![](https://ww1.sinaimg.cn/large/007i4MEmly1g1l5jjstuug30go0chgoy.gif)

下面我们由易到难一步步实现这个`SelfVue`。

我们只实现简易版的`vue`过程，主要包括**双括号**和**`v-model`** 和事件指令的功能。

### Object.defineProperty()

首先我们在引入`vue`文件之后在控制台打印一下`vue`实例中的`data`数据：

```javascript
var vm = new Vue({
    data: {
        obj: {
            a: 1
        }
    },
    created: function () {
        console.log(this.obj);
    }
});
```

![](https://ww1.sinaimg.cn/large/007i4MEmly1g1l5m5730qj30j6094gmt.jpg)

可见这是一个拥有`get`和`set`方法的对象。因为`Vue`是通过`Object.defineProperty()`来实现数据劫持的。

#### 三个参数

- `obj`：必须，目标对象
- `prop`：必须，需定义或修改的属性的名字
- `descriptor`：必须，目标属性所拥有的特性
  - 可以提供两种形式的设置：**数据描述** & **存取器描述**



**该函数返回设置完毕后的参数对象**



所以我们可以通过该属性设置它的存取器让其发扬我们自己的特性也就是常规操作啦

#### 举个栗子

我们现在有一个对象`Book`

```javascript
var Book = {
  name: 'vue权威指南'
};
console.log(Book.name);  // vue权威指南
```

如果我们想要在执行`console.log(book.name)`的同时，直接给书名加个书名号，或者说要通过什么监听对象`Book`的属性值。这时候`Object.defineProperty()`就派上用场了

```javascript
var Book = {}
var name = '';
Object.defineProperty(Book, 'name', {
  set: function (value) {
    name = value;
    console.log('你取了一个书名叫做' + value);
  },
  get: function () {
    return '《' + name + '》'
  }
})
 
Book.name = 'vue权威指南';  // 你取了一个书名叫做vue权威指南
console.log(Book.name);  // 《vue权威指南》
```

喏， 咱们现在的`get`和`set`就是自己定义的啦。

还记不记得刚才咱们打印的`Vue`实例？现在咱们也打印一下这个对象（`console.log(Book)`）

![](https://0d077ef9e74d8.cdn.sohucs.com/rmd0ZU7_jpg)

所以可以确定，**`Vue`确实是通过此种方式对数据进行劫持的**

### 思路分析

双向绑定的思路就是两个方面：

- 数据变化更新视图
- 视图变化更新数据

![modelview-viewmodel](https://0d077ef9e74d8.cdn.sohucs.com/rmd1iTT_jpg)

前边咱们讲`Object.defineProperty`的意思就是——我们可以通过改变数据来更新视图——只需要在相应的`set`函数中添加对应的 DOM 操作就可以啦



我们只需要给要监听的对象（`Watcher`）设置一个`set`函数，当数据改变自然会触发。我们将更新所需的方法放在其中就可以实现啦

![监听器](https://0d077ef9e74d8.cdn.sohucs.com/rmd1tYI_jpg)

### 实现过程

我们所需要的流程如下

![vue实现流程](https://0d077ef9e74d8.cdn.sohucs.com/rmd1HWt_png)

**需要三个身份：**

- 监听者（`Observer`）：用来劫持并监听所有属性。若有变动则同之订阅者
- 订阅者（`Watcher`）：可以收到属性的变化通知并执行相应的函数，从而更新视图
- 解析器（`Compile`）：可以扫描和解析每个节点的相关指令，并根据初始化模板数据初始化相应的订阅器

#### Observer实现

之前我们提到了`Observer`是一个数据监听器，其核心方法就是前文所述的`Object.defineProperty()`。我们可以通过递归的方式遍历监听所有属性值，并对其用`Object.defineProperty`处理相应操作

```javascript
function defineReactive(data, key, val) {
    observe(val); // 递归遍历所有子属性
    Object.defineProperty(data, key, {
        enumerable: true,
        configurable: true,
        get: function() {
            return val;
        },
        set: function(newVal) {
            val = newVal;
            console.log('属性' + key + '已经被监听了，现在值为：“' + newVal.toString() + '”');
        }
    });
}
 
function observe(data) {
    if (!data || typeof data !== 'object') {
        return;
    }
    Object.keys(data).forEach(function(key) {
        defineReactive(data, key, data[key]);
    });
};
 
var library = {
    book1: {
        name: ''
    },
    book2: ''
};
observe(library);
library.book1.name = 'vue权威指南'; // 属性name已经被监听了，现在值为：“vue权威指南”
library.book2 = '没有此书籍';  // 属性book2已经被监听了，现在值为：“没有此书籍”
```

订阅者（`Watcher`）显然不止一个，所以我们需要有一个用来容纳订阅者的容器——**消息订阅器（`Dep`）**。

> `Dep`主要负责收集订阅者，然后在属性变化之时执行相应`Watcher`的更新函数

为此我们需要给`Dep`一个容器——`list`

同时我们需要将`Observer`改造一下，植入消息订阅器

```javascript
function defineReactive(data, key, val) {
    observe(val); // 递归遍历所有子属性
    var dep = new Dep(); 
    Object.defineProperty(data, key, {
        enumerable: true,
        configurable: true,
        get: function() {
            if (是否需要添加订阅者) {
                dep.addSub(watcher); // 在这里添加一个订阅者
            }
            return val;
        },
        set: function(newVal) {
            if (val === newVal) {
                return;
            }
            val = newVal;
            console.log('属性' + key + '已经被监听了，现在值为：“' + newVal.toString() + '”');
            dep.notify(); // 如果数据变化，通知所有订阅者
        }
    });
}
 
function Dep () {
    this.subs = [];
}
Dep.prototype = {
    addSub: function(sub) {
        this.subs.push(sub);
    },
    notify: function() {
        this.subs.forEach(function(sub) {
            sub.update();
        });
    }
};
```

我们将订阅器`Dep`添加订阅者的代码块放在`getter`中，这是为了让`Watcher`初始化进行触发，**所以需要判断是否要添加订阅者。**

而`setter`则负责变化的数据通知给订阅者，令其执行相应的操作。当然，订阅者的`update`函数咱们一会儿再实现。

#### Watcher实现

`Watcher`在初始化的时候就需要将自己添加到订阅器`Dep`中。

已知监听器（`Observer`）是在`get`函数添加了订阅者（`Watcher`）之后工作的，所以我们只需要在订阅者（`Watcher`）初始化的时候触发对应的`get`函数区去执行添加订阅者操作即可



如何触发订阅者（`Watcher`）的`get`函数？

当然是获取其相应的属性值



另外还有一个细节点需要处理：我们只要在订阅者（`Watcher`）初始化的时候才需要添加订阅者，所以可以在订阅器`Dep`上做点手脚：**在`Dep.target`上缓存下订阅者，添加成功后再将其去掉**

```javascript
function Watcher(vm, exp, cb) {
    this.cb = cb;
    this.vm = vm;
    this.exp = exp;
    this.value = this.get();  // 将自己添加到订阅器的操作
}
 
Watcher.prototype = {
    update: function() {
        this.run();
    },
    run: function() {
        var value = this.vm.data[this.exp];
        var oldVal = this.value;
        if (value !== oldVal) {
            this.value = value;
            this.cb.call(this.vm, value, oldVal);
        }
    },
    get: function() {
        Dep.target = this;  // 缓存自己
        var value = this.vm.data[this.exp]  // 强制执行监听器里的get函数
        Dep.target = null;  // 释放自己
        return value;
    }
};
```

此时为了兼容新加入的订阅者`Watcher`我们需要给监听器`Observer`做个微调：

设置缓存

```javascript
function defineReactive(data, key, val) {
    observe(val); // 递归遍历所有子属性
    var dep = new Dep(); 
    Object.defineProperty(data, key, {
        enumerable: true,
        configurable: true,
        get: function() {
            if (Dep.target) {.  // 判断是否需要添加订阅者
                dep.addSub(Dep.target); // 在这里添加一个订阅者
            }
            return val;
        },
        set: function(newVal) {
            if (val === newVal) {
                return;
            }
            val = newVal;
            console.log('属性' + key + '已经被监听了，现在值为：“' + newVal.toString() + '”');
            dep.notify(); // 如果数据变化，通知所有订阅者
        }
    });
}
Dep.target = null;
```

至此我们就可以进行一个简单的双向绑定数据啦。因为此处没有解析器`Compile`所以对于模板数据我们进行写死处理。

```html
<body>
    <h1 id="name">{{name}}</h1>
</body>
```

将监听者`Observer`和订阅者`Watcher`关联起来：

```javascript
function SelfVue (data, el, exp) {
    this.data = data;
    observe(data);
    el.innerHTML = this.data[exp];  // 初始化模板数据的值
    new Watcher(this, exp, function (value) {
        el.innerHTML = value;
    });
    return this;
}
```

然后在页面上new一下`SelfVue`类，就可以实现数据的双向绑定了。我们在页面上试试看

```html
<body>
    <h1 id="name">{{name}}</h1>
</body>
<script src="js/observer.js"></script>
<script src="js/watcher.js"></script>
<script src="js/index.js"></script>
<script type="text/javascript">
    var ele = document.querySelector('#name');
    var selfVue = new SelfVue({
        name: 'hello world'
    }, ele, 'name');
 
    window.setTimeout(function () {
        console.log('name值改变了');
        selfVue.data.name = 'canfoo';
    }, 2000);
 
</script>
```

此时打开页面，可以看到刚开始页面显示`'hello world'`，2s之后变成了`'canfoo'`。

但此处有一个问题：我们每次赋值时需要这样：`seleVue.data.name = yourName`。为了让咱们体验好一点（更加趋近`Vue`），我们可以在进行`new SelfVue`时对其做一个代理，让访问`selfVue`的属性代理访问`selfVue.data`的属性

思路还是一样，用`Object.defineProperty()`包装

```javascript
function SelfVue (data, el, exp) {
    var self = this;
    this.data = data;
 
    Object.keys(data).forEach(function(key) {
        self.proxyKeys(key);  // 绑定代理属性
    });
 
    observe(data);
    el.innerHTML = this.data[exp];  // 初始化模板数据的值
    new Watcher(this, exp, function (value) {
        el.innerHTML = value;
    });
    return this;
}
 
SelfVue.prototype = {
    proxyKeys: function (key) {
        var self = this;
        Object.defineProperty(this, key, {
            enumerable: false,
            configurable: true,
            get: function proxyGetter() {
                return self.data[key];
            },
            set: function proxySetter(newVal) {
                self.data[key] = newVal;
            }
        });
    }
}
```

#### Compile实现

上边虽然已经实现了双向绑定，但是本质上是个阉割版的，只能监听固定的节点。所以我们现在试着搞一个解析器`Compile`来进行**解析**和**绑定**工作

**实现步骤**

- 解析模板指令，并替换模板数据，初始化视图
- 将模板指令对应的节点绑定对应的更新函数，初始化相应的订阅器



为了解析模板，首先需要获取到 DOM 元素，然后对 DOM 元素上含有指令的节点进行处理。为了避免对 DOM 的频繁操作，我们可以先建立一个`fragment`片段，将需要解析的 DOM 节点存入`fragment`片段再进行处理

```javascript
// 缓存DOM处理的fragment片段
function nodeToFragment (el) {
    var fragment = document.createDocumentFragment();
    var child = el.firstChild;
    while (child) {
        // 将Dom元素移入fragment中
        fragment.appendChild(child);
        child = el.firstChild
    }
    return fragment;
}
```

接下来就是遍历各个节点，对含有相关指令的节点进行特殊处理。

##### 双括号

```javascript
function compileElement (el) {
    var childNodes = el.childNodes;
    var self = this;
    [].slice.call(childNodes).forEach(function(node) {
        var reg = /\{\{(.*)\}\}/;
        var text = node.textContent;
 
        if (self.isTextNode(node) && reg.test(text)) {  // 判断是否是符合这种形式{{}}的指令
            self.compileText(node, reg.exec(text)[1]);
        }
 
        if (node.childNodes && node.childNodes.length) {
            self.compileElement(node);  // 继续递归遍历子节点
        }
    });
},
function compileText (node, exp) {
    var self = this;
    var initText = this.vm[exp];
    updateText(node, initText);  // 将初始化的数据初始化到视图中
    new Watcher(this.vm, exp, function (value) {  // 生成订阅器并绑定更新函数
        self.updateText(node, value);
    });
},
function updateText (node, value) {
    node.textContent = typeof value == 'undefined' ? '' : value;
}
```

获取到最外层节点后，调用`compileElement`函数，对所有子节点进行判断。若节点为文本节点且匹配**双括号**，则这种形式的指令就开始进行编译处理。

- 编译处理首先需要初始化视图数据（解析模板指令，并替换模板数据，初始化视图）
- 接下来需要生成一个订阅者`Watcher`并绑定更新函数的订阅器（将模板指令对应的节点绑定对应的更新函数，初始化相应的订阅器）

为了将解析器`Compile`与监听器`Observer`和订阅者`Watcher`关联起来，我们需要修改一下类`SelfVue`函数

```javascript
function SelfVue (options) {
    var self = this;
    this.vm = this;
    this.data = options;
 
    Object.keys(this.data).forEach(function(key) {
        self.proxyKeys(key);
    });
 
    observe(this.data);
    new Compile(options, this.vm);
    return this;
}
```

更改后，我们就不要像之前通过传入固定的元素值进行双向绑定了，可以随便命名各种变量进行双向绑定

```html
<body>
    <div id="app">
        <h2>{{title}}</h2>
        <h1>{{name}}</h1>
    </div>
</body>
<script src="js/observer.js"></script>
<script src="js/watcher.js"></script>
<script src="js/compile.js"></script>
<script src="js/index.js"></script>
<script type="text/javascript">
 
    var selfVue = new SelfVue({
        el: '#app',
        data: {
            title: 'hello world',
            name: ''
        }
    });
 
    window.setTimeout(function () {
        selfVue.title = '你好';
    }, 2000);
 
    window.setTimeout(function () {
        selfVue.name = 'canfoo';
    }, 2500);
 
</script>
```

以上，我们可以观察到，刚开始`title`和`name`分别被初始化为`hello world`和空，2s 后`title`被替换为`'你好'`，3s 后`name`被替换为`'canfoo'`。

此时我们已经完成了双向绑定的第一个功能：解析双括号

##### 添加一个v-model

到此为止我们只是实现了解析器`Compile`的其中一个基本的双向绑定功能，而现在我们准备向着更远的地方行进——完善更多指令的解析编译

至于方式嘛~很简单，我们继续在`compileElement`函数上对其他指令节点进行判断，然后遍历其所有属性，看是否有匹配的指令的属性。若有则对其进行解析编译。

现在我们实现一个`v-model`指令和事件指令的解析编译，对于这些节点我们使用`compile`函数进行解析处理

```javascript
function compile (node) {
    var nodeAttrs = node.attributes;
    var self = this;
    Array.prototype.forEach.call(nodeAttrs, function(attr) {
        var attrName = attr.name;
        if (self.isDirective(attrName)) {
            var exp = attr.value;
            var dir = attrName.substring(2);
            if (self.isEventDirective(dir)) {  // 事件指令
                self.compileEvent(node, self.vm, exp, dir);
            } else {  // v-model 指令
                self.compileModel(node, self.vm, exp, dir);
            }
            node.removeAttribute(attrName);
        }
    });
}
```

`compile`函数是挂在`Compile`原型上的。它首先遍历所有的节点属性，再判断属性是否是指令属性。若是则再区分是哪种指令，然后再做相应处理。

最后稍微改造一下`SelfVue`类，使其更趋近`vue`的用法

```javascript
function SelfVue (options) {
    var self = this;
    this.data = options.data;
    this.methods = options.methods;
 
    Object.keys(this.data).forEach(function(key) {
        self.proxyKeys(key);
    });
 
    observe(this.data);
    new Compile(options.el, this);
    options.mounted.call(this); // 所有事情处理好后执行mounted函数
}
```

我们来试试看

```html
<body>
    <div id="app">
        <h2>{{title}}</h2>
        <input v-model="name">
        <h1>{{name}}</h1>
        <button v-on:click="clickMe">click me!</button>
    </div>
</body>
<script src="js/observer.js"></script>
<script src="js/watcher.js"></script>
<script src="js/compile.js"></script>
<script src="js/index.js"></script>
<script type="text/javascript">
 
     new SelfVue({
        el: '#app',
        data: {
            title: 'hello world',
            name: 'canfoo'
        },
        methods: {
            clickMe: function () {
                this.title = 'hello world';
            }
        },
        mounted: function () {
            window.setTimeout(() => {
                this.title = '你好';
            }, 1000);
        }
    });
 
</script>
```

结果如下

![展示效果](https://ww1.sinaimg.cn/large/007i4MEmly1g1l5jjstuug30go0chgoy.gif)

### last

本文转载自博客园的`canfoo`，[原文地址](<https://www.cnblogs.com/canfoo/p/6891868.html>)

在此表示对作者深深的敬意。