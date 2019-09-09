---
title: Object.create()
categories: ES6
tags: 
 - Object.create()
 - 原型继承
---

在将 ES6 中的 Class 继承时我们会用到`Object.create()`方法。所以作为铺垫我们先把这个`Object`方法了解一番

<!--more-->

### 从一个栗子开始

讲解之前我们先看一段 ES6 实现继承的代码

```javascript
class Point {
    static NAME='point';

    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
  
    outPoint(){
        console.log('point:'+this.x+this.y);
    }
}

class ColorPoint extends Point {
    static COLOR_NAME='ColorPoint';

    constructor(x, y, color) {
        super(x, y);    
        this.color = color;
    }

    outColorPoint(){
        console.log('ColorPoint:'+this.x+this.y+this.color);
    }
}
```

通过`babel`将其转为 ES5

```javascript
'use strict';

var _createClass = function() {
    function defineProperties(target, props) {
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            if ("value" in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor);
        }
    }
    return function(Constructor, protoProps, staticProps) {
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
} ();

function _possibleConstructorReturn(self, call) {
    if (!self) {
        throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
    }
    return call && (typeof call === "object" || typeof call === "function") ? call: self;
}

function _inherits(subClass, superClass) {
    if (typeof superClass !== "function" && superClass !== null) {
        throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
    }
    subClass.prototype = Object.create(superClass && superClass.prototype, {
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
}

function _classCallCheck(instance, Constructor) {
    if (! (instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}

var Point = function() {
    function Point(x, y) {
        _classCallCheck(this, Point);

        this.x = x;
        this.y = y;
    }

    _createClass(Point, [{
        key: 'outPoint',
        value: function outPoint() {
            console.log('point:' + this.x + this.y);
        }
    }]);

    return Point;
} ();

Point.NAME = 'point';

var ColorPoint = function(_Point) {
    _inherits(ColorPoint, _Point);

    function ColorPoint(x, y, color) {
        _classCallCheck(this, ColorPoint);

        var _this = _possibleConstructorReturn(this, (ColorPoint.__proto__ || Object.getPrototypeOf(ColorPoint)).call(this, x, y));

        _this.color = color;
        return _this;
    }

    _createClass(ColorPoint, [{
        key: 'outColorPoint',
        value: function outColorPoint() {
            console.log('ColorPoint:' + this.x + this.y + this.color);
        }
    }]);

    return ColorPoint;
} (Point);

ColorPoint.COLOR_NAME = 'ColorPoint';
```

这个有点复杂，所以咱们试着将代码简化，去掉验证信息，`_createClass`用`prototype`代替

```javascript
'use strict';
// 实现父类原型对象的继承
function _inherits(subClass, superClass) {
    subClass.prototype = Object.create(superClass.prototype, {
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    subClass.__proto__ = superClass;
}
var Point = function(x, y) {
    this.x = x;
    this.y = y;
}
Point.prototype.outPoint = function outPoint() {
    console.log('point:' + this.x + this.y);
}

Point.NAME = 'point';

var ColorPoint = function(_Point) {
    _inherits(ColorPoint, _Point);

    function ColorPoint(x, y, color) {
        ColorPoint.__proto__.call(this,x,y);
        this.color = color;
    }
    ColorPoint.prototype.outColorPoint = function() {
        console.log('ColorPoint:' + this.x + this.y + this.color);
    }
    return ColorPoint;
} (Point);

ColorPoint.COLOR_NAME = 'ColorPoint';    
```

可见 ES6 中的类继承具备以下几个特点：

- 处于严格模式下，给各种属性变量多一层保障（防止变量提升）
- 子类通过**原型链继承**继承父类的方法，通过**构造函数式继承**继承父类的属性（防止子类实例间相互污染）

### Object.create

也许这个时候`Object.create`会问：

![](https://pic.superbed.cn/item/5c986bc43a213b041711547e)

别急，咱们慢慢儿来

你刚才肯定没仔细看上边的代码。咱们单拎出来，是不是有一句`subClass.prototype = Object.create(superClass.prototype, {...})`

```javascript
function _inherits(subClass, superClass) {
    subClass.prototype = Object.create(superClass.prototype, {
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    subClass.__proto__ = superClass;
}
```

这东西是用来实现对父类原型对象的继承的。

![](https://pic.superbed.cn/item/5c9870be3a213b041711c762)

瞅瞅，通过`Object.create()`B继承了A的所有属性和方法

所以这东西可以用下面的代码代替

```javascript
Object.create = function (obj) {
  function F() {}
  F.prototype = obj;
  return new F();
};
```

不信我给你演示一下

```javascript
Object.prototype.inher = function(obj){
	function F(){}
	F.prototype = obj;
	return new F();
}
var A = {
	a: 1,
	print: function(){
		console.log('hello world')
	}
}
var B = Object.inher(A)

// 打印
B
```



![](https://pic.superbed.cn/item/5c9879963a213b041712534c)

是不是跟上边的一模一样？

不错，`Object.create()`的实质正是**原型式继承**。（中间定义了一个干净的过度类，相当于对类式继承的一种封装）

**这东西的属性更改不会造成父属性的变化**，还蛮好用的

```javascript
Object.prototype.inher = function(obj){
	function F(){}
	F.prototype = obj;
	return new F();
}
var A = {
	a: 1,
	print: function(){
		console.log('hello world')
	}
}
var B = Object.inher(A)
B.a = 2
var C = Object.inher(B)

// 打印
A
B
C
```

A、B、C打印结果如下

![](https://pic.superbed.cn/item/5c987b6f3a213b04171293b1)

看到了没，我们更改`B`的`a`属性时`A`的`a`属性并未跟着改变。

所以**Object.create() 完成的继承在进行属性修改时不会造成父属性的变化**。

### 继承

所以，如果我们希望继承某个类/对象则可以直接使用`Object.create(class/obj)`的方法。

如此我们即可动态继承原型的所有属性和方法。



当然，如果我们想要生成一个不继承任何属性（比如没有`toString`和`valueOf`方法）的对象，可以将`Object.create`的参数设为`null`。

```javascript
var obj = Object.create(null);
```

#### 继承的实际是原型

```javascript
var obj1 = { p: 1 };
var obj2 = Object.create(obj1);

obj1.p = 2;
obj2.p // 2
```

这一点咱们也可以验证一下

![](https://pic.superbed.cn/item/5c987ce43a213b041712a1da)

**所以他本质上和原型链继承一样**，修改原型对象`C`会影响到原型对象`D`。（因为是原型链继承所以`B`修改后并未开辟新的存储空间，只是动态的更新了属性`a`的值）

### 第二个参数？

没错。这东西好用就在这里。我们可以通过此方法的第二个参数手动设置子类的对象属性。

```javascript
var obj = Object.create({}, {
  p1: {
    value: 123,
    enumerable: true,
    configurable: true,
    writable: true,
  },
  p2: {
    value: 'abc',
    enumerable: true,
    configurable: true,
    writable: true,
  }
});

// 等同于
var obj = Object.create({});
obj.p1 = 123;
obj.p2 = 'abc';
```

如此一来我们就可以`DIY`子类啦。

我们今后的继承（原型继承）我们就可以通过它来实现。同时下一章的`Class`语法我们也会用到这个方法。

那么，后会有期

![](<https://ww1.sinaimg.cn/large/007i4MEmgy1g1avrg3n5gj30kq0kqq3j.jpg>)