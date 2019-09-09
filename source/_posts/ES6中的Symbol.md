---
title: ES6中的Symbol
categories: ES6
---

我们今天一起看一些 ES6 新增的基本类型`Symbol`

<!--more-->

- `Symbol`函数前不能使用`new`命令，否则会报错。这是因为生成的 Symbol 是一个原始类型的值，不是对象。也就是说，由于 Symbol 值不是对象，所以不能添加属性。基本上，它是一种**类似于字符串的数据类型。**

- `Symbol`函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。

- `Symbol`函数的参数只是表示对当前 Symbol 值的描述，因此相同参数的`Symbol`函数的返回值是不相等的。

- `Symbol`值不能与其他类型的值进行运算，会报错。

- `Symbol`值也可以转为布尔值，但是不能转为数值。

- **`Symbol`值一般用作属性名，以防止某个键被覆盖或改写。**

  - ```javascript
    let mySymbol = Symbol();
    
    // 第一种写法
    let a = {};
    a[mySymbol] = 'Hello!';
    
    // 第二种写法
    let a = {
      [mySymbol]: 'Hello!'
    };
    
    // 第三种写法
    let a = {};
    Object.defineProperty(a, mySymbol, { value: 'Hello!' });
    
    // 以上写法都得到同样结果
    a[mySymbol] // "Hello!"
    ```

  - 但是`Symbol`值作为对象的属性名时不能用点运算符：

    ```javascript
    const mySymbol = Symbol();
    const a = {};
    
    a.mySymbol = 'Hello!';
    a[mySymbol] // undefined
    a['mySymbol'] // "Hello!"
    ```

- 在对象内部使用`Symbol`值定义属性时，`Symbol`值必须放在方括号中

  - ```javascript
    let s = Symbol();
    
    let obj = {
      [s]: function (arg) { ... }	// 或是对象增强写法： [s](arg) { ... }
    };
    
    obj[s](123);
    ```

<!--more-->

### 属性名的遍历

作为属性名的`Symbol`值和其他字符串类型的键不同，无法通过`for...in`、`for...of`循环中，也不会被`Object.keys()`、`Object.getOwnPropertyNames()`、`JSON.stringify()`返回。

**它通过`Object.getOwnPropertySymbols`方法返回指定对象的所有`Symbol`属性名（返回一个数组）**

故<u>通过此方法可以为对象定义一些非私有的、但又希望只用于内部的方法</u>

```javascript
let size = Symbol('size');

class Collection {
  constructor() {
    this[size] = 0;
  }

  add(item) {
    this[this[size]] = item;
    this[size]++;
  }

  static sizeOf(instance) {
    return instance[size];
  }
}

let x = new Collection();
Collection.sizeOf(x) // 0

x.add('foo');
Collection.sizeOf(x) // 1

Object.keys(x) // ['0']
Object.getOwnPropertyNames(x) // ['0']
Object.getOwnPropertySymbols(x) // [Symbol(size)]
```



另一个新的 API，**`Reflect.ownKeys`方法可以返回所有类型的键名，包括常规键名和 Symbol 键名**。

### 模块的Singleton模式

`Singleton`模式指的是调用一个类，任何时候返回的都是同一个实例。

对于`node`来讲，模块文件可以看作是一个类，为了保证每次执行此模块文件返回的都是同一个实例，我们常把实例放到顶层对象`global`

```javascript
// mod.js
function A() {
  this.foo = 'hello';
}

if (!global._foo) {
  global._foo = new A();
}

module.exports = global._foo;
```

然后加载上边的`mod.js`

```javascript
const a = require('./mod.js');
console.log(a.foo);
```



**但问题是我们必须令引用`mod.js`的诸多文件既可以使用，又无法修改**



为防止此情况的发生，我们可以使用`Symbol`

```javascript
// mod.js
const FOO_KEY = Symbol('foo');

function A() {
  this.foo = 'hello';
}

if (!global[FOO_KEY]) {
  global[FOO_KEY] = new A();
}

module.exports = global[FOO_KEY];
```

若键名使用`Symbol`方法生成，那么外部将无法引用这个值，自然也无法改写。