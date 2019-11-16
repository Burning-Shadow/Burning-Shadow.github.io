---
title: JavaScript中的prototype、[[prototype]]和__proto__的区别和用法
date: 2019-04-29 15:21:00
categories: JavaScript —— 原理篇
tags: 
- __proto__
- prototype
- 原型
- 原型链
---

### 显式原型&隐式原型
显式原型：`prototype`
隐式原型：`__proto__`

#### Important
- `__proto__`是每个对象都具有的属性
- `prototype`是`Function`独有的属性

<!--more-->

#### Tips

- 对象的隐式原型的值为其对应构造函数的显式原型的值
  - `fn.__proto__ === Function.prototype`
- 函数的`prototype`属性是定义时自动添加的。默认为`{}`
- 对象的`__proto__`属性是创建对象时自动添加的，默认值为其构造函数的`prototype`
- `Object.prototype.__proto__ === null`


#### 说了这么多和`[[prototype]]`有什么关系?
其实`[[prototype]]`和`__proto__`意义相同，均表示对象的内部属性，其值指向对象原型。前者在一些书籍、规范中表示一个对象的原型属性，后者则是在浏览器实现中指向对象原型。


### 作用
作用方面来讲当然是实现继承了。其中最经典的共享属性方法的原型链继承。其中必不可少的属性就是`__protoo__`和`prototype`。

我们举个栗子
```javascript
function Son(){}
function Father(){}
Son.prototype = new Father();
```
如此即实现了继承
我们可以写代码进行验证
```javascript
Son.prototype.__proto__ === Father.prototype  //true
```
符合上述第三条规则，所以可以通过此方法完成函数的继承。
**注意，此为原型链继承，其中的方法鱼属性为此链上的所有实例所共享**

### 教你手撸原型链
原型链无非就是一堆继承关系：
- 我们只需要将儿子原型的`__proto__`属性指向父亲的`prototype`属性，构造函数的`prototype`属性的`constructor`属性指向其本身即可。
- 不过需要注意的一点是，**已经被实例化的对象**`__proto__`属性指向其构造函数的`prototype`。
- 另外一个特殊的对象`Object`。作为所有对象的父类他的原型的`__proto__`属性指向`null`

![在这里插入图片描述](https://pic.superbed.cn/item/5c93bda83a213b0417da42a9)
如此我们就可以看得懂那张经典的原型链图解啦

### 从原理讨论原型链的用处
#### typeof
`typeof`作为《JavaScript高级程序设计》中首推的判断类型方法无疑是大多数人的选择。不过令人烦恼的是我们发现当碰到`Array`、`Function`等类型时他均返回一个`Object`。这就有点气人了。所以机智的玩家们采取了`Object.prototype.toString.call(obj)`方法来识别对象类型。他会返回一个`"[object Type]"`的东西来告诉你所指对象的类型。

此处就用到了原型链必不可少的`prototype`。通过改变`this`指针指向将我们所要验证类型的对象。以完成类型的检验
#### instanceof
除去`typeof`外我们还有另一个方法：`instanceof`

看过我之前转载的关于typeof与instanceof原理的好兄弟估计会猜到我要说什么,不过我还是要继续BB。

`instanceof`这个方法用于判断某实例是否从属于某种类型，同时也可以判断一个实例是否是其父类型或者祖先类型的实例。举个栗子
```javascript
function Son(){}
function Father(){}
Son.prototype = new Father();

var son = new Son();
son instanceof Son  //true
son instanceof Father  //true
```
看见没，只要你是这一家子的人（实例）那你无论走哪这祖宗祠堂（原型链上的老东西们）都认你。那么他们是如何判断你是不是这家的儿孙呢？当然是原型链。我们来试着写一串伪代码来进行验证
```javascript
function new_instance_of(leftVaule, rightVaule) { 
    let rightProto = rightVaule.prototype; // 取右表达式的 prototype 值
    leftVaule = leftVaule.__proto__; // 取左表达式的__proto__值
    while (true) {
    	if (leftVaule === null) {
            return false;	
        }
        if (leftVaule === rightProto) {
            return true;	
        } 
        leftVaule = leftVaule.__proto__ 
    }
}
```
如此这般，通过原型链认祖归宗。咱们看看是否有用
```javascript
new_instance_of(son, Son)	// true
new_instance_of(son, Father)	// true
```



### 构造函数创建对象实例

**JavaScript** 函数有两个不同的内部方法：**[[Call]]** 和 **[[Construct]]** 。

如果不通过`new`关键字调用函数，则执行 **[[Call]]** 函数，从而直接执行代码中的函数体。

当通过`new`关键字调用函数时，执行的是 **[[Construct]]** 函数，它负责创建一个实例对象，把实例对象的`__proto__`属性指向构造函数的`prototype`来实现继承构造函数`prototype`的所有属性和方法，将`this`绑定到实例上，然后再执行函数体。

模拟一个构造函数：

```javascript
// 传入的是函数的原型
function createObject(proto) {
    if (!(proto === null || typeof proto === "object" || typeof proto === "function"){
        throw TypeError('Argument must be an object, or null');
    }
    var obj = new Object();
    obj.__proto__ = proto;
    return obj;
}

var foo = createObject(Foo.prototype);
```





### 最后唠两句

上边那张原型链的图想必大家也可以手撸了。但是作为构造函数的基类，`Function`还和其他引用类型不太一样

![](https://pic.superbed.cn/item/5c94950c3a213b0417e18b36)

如图，我们在控制台输入`Foo.__proto__ === Function.prototype`后打印结果为`true`。所以我们可以理解为 **所有函数都是`Function`的实例。**







今天的内容就这么多
![在这里插入图片描述](https://0d077ef9e74d8.cdn.sohucs.com/rln2I4a_jpg)