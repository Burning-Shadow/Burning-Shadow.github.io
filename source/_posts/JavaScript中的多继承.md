---
title: JavaScript中的多继承&多态
categories: JavaScript设计模式
tags: 
 - 多继承
 - 多态
---

理论上讲，`JavaScript`中继承是依赖于原型`prototype`链实现继承的。由于只有一条链所以理论上讲无法实现多继承。但是我们可以另辟蹊径

<!--more-->

### 多继承

#### 先从继承单对象属性说起

```javascript
// 单继承 属性复制
var extend = function(target, source){
    // 遍历源对象的属性
    for(var property in source){
		// 将源对象中的属性复制到目标对象中
        target[property] = source[property]
    }
   	return target
}
```

这是对对象中属性的一个浅复制的过程，同样的，继承也是一样的原理。

#### 实现

所以既然无法通过原型链完成多继承，那么我们就通过对其属性的复制来完成继承

```javascript
var mix = function(){
    var i=1,
        length = arguments.length,
        target = arguments[0],
        arg;						// 缓存参数对象
    
    // 遍历被继承的对象
    for( ;i<len; i++){
        arg = arguments[i]
        for(var property in arg){
            target[property] = arg[property]
        }
    }
    return target
}
```

最后说明一点，由于我们本身只是对概念的一个论述，所以就只讲浅复制了。今后我闲下来没事儿干的时候再码一下深复制的代码

### 多态

多态，顾名思义，同于i个方法具有多种调用方式。

在`JavaScript`中若想实现多态我们需要对传入的参数加以判别，以此来实现`OOP`中的多态。

我们举个栗子：根据传入参数个数的不同来返回不同的结果——传入一个参数则返回其加10后的值，传入两个参数则返回二者之和

```javascript
function Add(){
    // 无参数
    function zero(){
        return 10
    }
    
    // 一个参数
    function one(num){
        return num+10
    }
    
    // 两个参数
    function two(num1, num2){
        return num1+num2
    }
    
    // 相加公有方法
    this.add = function(){
        var arg = arguments,
            len = arg.length;
        
        switch(len){
            case 0:
                return zero();
            case 1:
                return one(arg[0]);
            case 2:
                return two(arg[0], arg[1])
        }
    }
}

// 实例化类
var A = new Add()

console.log(A.add())		// 10
console.log(A.add(1))		// 11
console.log(A.add(1, 2))	// 3
```

