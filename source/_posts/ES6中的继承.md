---
title: ES6中的继承
date: 2019-04-30 17:12:00
categories: ES6
tags:
 - extends
 - tags
---

ES6 中引入了新的关键字`extend`用以实现继承。算是之前对**寄生组合式继承的一个封装**。

<!--more-->

### 寄生组合式继承

首先我们来回顾一下寄生组合式继承

```javascript
function inheritPrototype(subType, superType){
  var prototype = Object.create(superType.prototype); // 创建对象，创建父类原型的一个副本
  prototype.constructor = subType;                    // 增强对象，弥补因重写原型而失去的默认的constructor 属性
  subType.prototype = prototype;                      // 指定对象，将新创建的对象赋值给子类的原型
}

// 父类初始化实例属性和原型属性
function SuperType(name){
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function(){
  alert(this.name);
};

// 借用构造函数传递增强子类实例属性（支持传参和避免篡改）
function SubType(name, age){
  SuperType.call(this, name);
  this.age = age;
}

// 将父类原型指向子类
inheritPrototype(SubType, SuperType);

// 新增子类原型属性
SubType.prototype.sayAge = function(){
  alert(this.age);
}

var instance1 = new SubType("xyc", 23);
var instance2 = new SubType("lxy", 23);

instance1.colors.push("2"); // ["red", "blue", "green", "2"]
instance1.colors.push("3"); // ["red", "blue", "green", "3"]

```

如图，精妙之处在于`Object.assign`。将所有可枚举属性的值从一个或多个源对象复制到目标对象，并将目标对象返回

- 子类继承了弗雷的属性和方法，同时，属性未被创建在原型链上，故不会共享同一属性
- 子类可以给父类动态传参
- 父类的构造函数只执行了一次

![](https://pic.superbed.cn/item/5cc7dfed3a213b04175b3cd0)

### Extends

`extends`关键字主要用于类声明或类表达式中，以创建一个类。该类是另一个类的子类。

其中，`constructor`表示构造函数。若未显式指定构造方法则会添加默认的`constructor`

```javascript
class Rectangle {
    // constructor
    constructor(height, width) {
        this.height = height;
        this.width = width;
    }
    
    // Getter
    get area() {
        return this.calcArea()
    }
    
    // Method
    calcArea() {
        return this.height * this.width;
    }
}

const rectangle = new Rectangle(10, 20);
console.log(rectangle.area);
// 输出 200

-----------------------------------------------------------------
// 继承
class Square extends Rectangle {

  constructor(length) {
    super(length, length);
    
    // 如果子类中存在构造函数，则需要在使用“this”之前首先调用 super()。
    this.name = 'Square';
  }

  get area() {
    return this.height * this.width;
  }
}

const square = new Square(10);
console.log(square.area);
// 输出 100

```

我们在控制台中打印一下`square`实例：

![](https://pic.superbed.cn/item/5cc80a273a213b04175d7684)

发现了没有，模式和上边的寄生组合式继承一模一样。

`height`和`width`是继承自父类的属性，`get area`则是从父类继承而来的方法。







其核心实现原理如下

```javascript
function _inherits(subType, superType) {
  
    // 创建对象，创建父类原型的一个副本
    // 增强对象，弥补因重写原型而失去的默认的constructor 属性
    // 指定对象，将新创建的对象赋值给子类的原型
    subType.prototype = Object.create(superType && superType.prototype, {
        constructor: {
            value: subType,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    
    if (superType) {
        Object.setPrototypeOf 
            ? Object.setPrototypeOf(subType, superType) 
            : subType.__proto__ = superType;
    }
}
```



### 总结

- 通过`class`关键字就可以声明一个类，而非像 ES5 中通过`function`定义
- `constructor`是构造器，相当于继承中的**构造函数式继承**，通过`this.attr`定义独属于本实例的属性（已达到互不干扰的效果）
- 方法则无需加上`function`关键字，直接`methodName(){...}`即可
- `extend`关键字用于连接父类和子类。`class SubType extends SuperType`
- 而在子类的构造器中则可以通过`super`关键字调用父类的构造函数，省去很多不便：`super(...args)`



