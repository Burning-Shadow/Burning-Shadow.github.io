---
title: ES6中的代理模式
date: 2019-04-12 15:53:00
categories: ES6
tags:
 - proxy
 - ES6
---

按照我们常规的语义理解，代理代理，代为受理，即起一个中间人的作用，代替雇主出面摆平，并可以做到一些原本雇主难以做到的事情。而 ES6 中的代理模式亦是如此，通过`Proxy`来完成对其他操作的控制。

<!--more-->

### 代理模式

`Proxy Pattern`是程序设计中的一种设计模式。而代理者是指一个类可以作为其他东西的接口....

还有其他一大堆鸟话，没看懂，所以就不贴上来了。咱们接着往下看吧

### Proxy 对象

> Proxy 对象用于定义基本操作的自定义行为（如属性查找，赋值，枚举，函数调用等）

什么意思呢？**`Proxy`对象就是可以让你去对`JavaScript`中的一切合法对象的基本操作进行自定义，然后用自己定义的操作去覆盖其对象的基本操作**

我们来看一下这东西：

```javascript
let proxy = new Proxy(target, handler)

// target: 你要代理的对象，可以是 JS 中任何合法对象
// handler: 你要自定义操作方法的一个集合
// proxy: 一个被代理后的新对象，拥有target的一切属性和方法。其行为和结果是在handler中定义的
```

知道基本操作之后我们举个实例

```javascript
let obj = {
  a: 1,
  b: 2,
}

const p = new Proxy(obj, {
  get(target, key, value) {
    if (key === 'c') {
      return '我是自定义的一个结果';
    } else {
      return target[key];
    }
  },

  set(target, key, value) {
    if (value === 4) {
      target[key] = '我是自定义的一个结果';
    } else {
      target[key] = value;
    }
  }
})

console.log(obj.a) // 1
console.log(obj.c) // undefined
console.log(p.a) // 1
console.log(p.c) // 我是自定义的一个结果

obj.name = '李白';
console.log(obj.name); // 李白
obj.age = 4;
console.log(obj.age); // 4

p.name = '李白';
console.log(p.name); // 李白
p.age = 4;
console.log(p.age); // 我是自定义的一个结果
```

有没有发现，贼鸡儿神奇！这东西可以对不存在的一些对象和变量进行操作！也就是说，`Proxy`对象的真正作用是**用于定义基本操作的自定义行为**。我们可以做什么呢？接着看8~

### handler 对象

我们在上一个例子中看到，`handler`中定义了`get`、`set`两个函数代替了对象原生所带的`get`和`set`方法。而事实上，`handler`对象设计的初衷就是为了自定义代理对象的各种代理操作。它一共有13种方法

| 方法                               | 作用                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| handler.getPrototypeOf()           | 在读取代理对象的原型时触发该操作，比如在执行 Object.getPrototypeOf(proxy) 时。 |
| handler.setPrototypeOf()           | 在设置代理对象的原型时触发该操作，比如在执行 Object.setPrototypeOf(proxy, null) 时。 |
| handler.isExtensible()             | 在判断一个代理对象是否是可扩展时触发该操作，比如在执行 Object.isExtensible(proxy) 时。 |
| handler.preventExtensions()        | 在让一个代理对象不可扩展时触发该操作，比如在执行 Object.preventExtensions(proxy) 时。 |
| handler.getOwnPropertyDescriptor() | 在获取代理对象某个属性的属性描述时触发该操作，比如在执行 Object.getOwnPropertyDescriptor(proxy, "foo") 时。 |
| handler.defineProperty()           | 在定义代理对象某个属性时的属性描述时触发该操作，比如在执行 Object.defineProperty(proxy, "foo", {}) 时。 |
| handler.has()                      | 在判断代理对象是否拥有某个属性时触发该操作，比如在执行 "foo" in proxy 时。 |
| handler.get()                      | 在读取代理对象的某个属性时触发该操作，比如在执行 proxy.foo 时。 |
| handler.set()                      | 在给代理对象的某个属性赋值时触发该操作，比如在执行 proxy.foo = 1 时。 |
| handler.deleteProperty()           | 在删除代理对象的某个属性时触发该操作，比如在执行 delete proxy.foo 时。 |
| handler.ownKeys()                  | 在获取代理对象的所有属性键时触发该操作，比如在执行 Object.getOwnPropertyNames(proxy) 时。 |
| handler.apply()                    | 在调用一个目标对象为函数的代理对象时触发该操作，比如在执行 proxy() 时。 |
| handler.construct()                | 在给一个目标对象为构造函数的代理对象构造实例时触发该操作，比如在执行new proxy() 时。 |

可以看到，基本上涵盖了所有我们所知道的对象的属性和操作方法。这也正是`Proxy`的强大之处

### 作用

说了这么多我们还不知道这东西咋用呢。

- 拦截和监视外部对对象的访问
- 降低函数或类的复杂度
- 在复杂操作前对操作进行校验或对所需资源进行管理



我们来举两个例子

#### 未定义属性报错

```
题目：对象中若我们打印未定义属性则会自动输出undefined。可是作为新手我们很难看出出错原因是因为未定义还是值确实为undefined。为此我们不妨将其用Proxy对象做一个代理，若是此对象确实未定义则报错，便于开发者纠错
```

别的不知道反正头条的面试是有过这道题的，作为一个初学者咱试着实现一下。

```javascript
var person = {
  name: "张三"
};

var proxy = new Proxy(person, {
  get: function(target, property) {
    if (property in target) {
      return target[property];
    } else {
      throw new ReferenceError("Property \"" + property + "\" does not exist.");
    }
  }
});
proxy.name	// 张三
proxy.age	// Uncaught ReferenceError: Property "age" does not exist.
```

当然我们也可以写的更加方便配置一些

```javascript
var person = {
  name: "张三"
};
handler = {
	get(target, property){
		if(property in target){
			return target[property]
		}else{
			throw new ReferenceError("Property \"" + property + "\" does not exist.")
		}
	}
}

var proxy = new Proxy(person, handler)

proxy.name	// 张三
proxy.age	// Uncaught ReferenceError: Property "age" does not exist.
```

而且最有意思的一点是我们通过`Proxy`进行的代理可以映射到实例上

```javascript
let proto = new Proxy({}, {
  get(target, propertyKey, receiver) {
    console.log('GET ' + propertyKey);
    return target[propertyKey];
  }
});

let obj = Object.create(proto);
obj.foo // "GET foo"
```

