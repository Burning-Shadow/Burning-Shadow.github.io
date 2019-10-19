---
title: 为什么Object.defineProperty不能检测到数组长度的变化
categories: JavaScript —— 原理篇
tags:
 - Object.defineProperty
---

昨天老袁面腾讯的时候被面试官从 Vue 的数据劫持跳转到这么个问题：**为什么Object.defineProperty不能检测到数组长度的变化？？？**

老袁面完后看起来很难过。我其实心里对这个问题也很纳闷：

![](https://pic.superbed.cn/item/5cbeda923a213b0417b93269)

<!--more-->

### 改变对象属性能检测到嘛？

由于数组属于引用类型，所以其本质上还是属于对象的。我们来看看对对象进行数据劫持后改变其子属性会有怎样的反应

```javascript
var obj = {
  name: {
    firstName: 'Bob',
    lastName: 'Liu'    
  }
}
Object.defineProperty(obj, name, {
  set: function (value) {
    name = value;
    console.log('你取了一个名字叫做' + value);
  },
  get: function () {
    return '你是' + name
  }
})
obj.name.firstName = 'Sir'			// "Sir"
obj.name.firstName					// "Sir"
```

咋样咋样，瞅着没，一个~~鸡巴~~德行！我子属性变化它是检测不到的！

那为什么检测不到呢？**有以下两种可能**

- 对象和数组作为引用类型之所以无法被检测到是因为我们存储在栈区的只是一个指向堆区的指针，数据的改变不会引起指向其指针的变化，所以无法被`Object.defineProperty`
- 对象和属性变化时分几种情况，当新增数据时由于属性名（索引）增加而无法被`Object.defineProperty`检测到所以无法通过`Objcet.defineProperty`监测数组变化。

**到底是哪种呢？**

### 铺垫

#### 属性类型

属性分为两种类型：**数据属性** & **访问器属性**

##### 数据属性

- `[[Configurable]]`：是否可配置，
  - 能否通过`delete`删除属性
  - 能否修改属性
  - 能否把属性修改为访问器属性
- `[[Enumerable]]`：能否通过`for-in`循环返回该属性
- `[[Get]]`：取值
- `[[Set]]`：赋值

##### 访问器属性

- `[[Configurable]]`：是否可配置，能否通过`delete`删除属性。
  - 能否通过`delete`删除属性
  - 能否修改属性
  - 能否把属性修改为访问器属性
- `[[Enumerable]]`：能否通过`for-in`循环返回该属性
- `[[Writable]]`：是否可写
- `[[Value]]`：属性的值

#### 属性创建的区别

我们平时通过`obj.attributeName`取值赋值时实际是修改其`[[Value]]`属性。而我们通过`Object.defineProperty`方式定义的属性对其通过`[[Get]]`和`[[Set]]`函数进行读写。

### 数组长度 & 索引

之所以我们无法通过`[[Get]]`和`[[Set]]`得知数组的更改，原因正是类似于上述的对象一般，`Object.defineProperty`无法检测到数组长度的变化。准确的说是无法检测到通过改变`length`而增加的长度

我们将数组的`length`属性初始化为：

```
enumberable: false
configurable: false
writable: true
```

即，无法删除和修改（并非赋值）`length`属性

```
Object.defineProperty(arr, 'length', { set(){}})
// Uncaught TypeError: Cannot redefine property: length
```

而数组索引则是访问数组值的一种方式。若拿它与对象相比较，**索引就是数组属性的`key`**，它与`length`是2个不同的概念

```javascript
var a = [a, b, c]
a.length = 10
// 只是显示的给length赋值，索引3-9的对应的value也会赋值undefined
// 但是索引3-9的key都是没有值的
// 我们可以用for-in打印，只会打印0,1,2
for (var key in a) {
  console.log(key) // 0,1,2
}
```

>JavaScript 数组的 length 属性和其数字下标之间有着紧密的联系。数组内置的几个方法（例如 join、slice、indexOf 等）都会考虑 length 的值。另外还有一些方法（例如 push、splice 等）还会改变 length 的值。

这些内置方法再操作数组时出去改变其中的内容还会影响`length`的值。分为两种情况

- 减少值

  - 当我们**`shift`一个数组时**你会发现它会遍历数组。此时**数组的索引对应的值得到了相应的更新**。这种情况可以被`Object.defineProperty`检测到，因为有属性（索引）的存在。

- 增加值

  - `push`值时，数组的长度会增加1，索引也会增加1.但此时的索引是新增的。虽然**`Object.defineProperty`不能检测到新增的属性(`push`之后`index`自增，相当于新增`key`)**，但是<u>在 Vue 中，新增的对象属性可以显式的调用`vm.$set`来添加监听</u>
  - **手动赋值`length`为一个更大的值。此时长度会更新，但对应的索引不会被赋值，即对象的属性为`null`**。`Object.defineProperty`再强也无法处理对未知属性的监听

  我们来看一下上面的论述

  ```javascript
  // 还是老套路，定义一个observe方法
  function defineReactive(data, key, val) {
    Object.defineProperty(data, key, {
      enumerable: true,
      configurable: true,
       get: function defineGet() {
        console.log(`get key: ${key} val: ${val}`)
        return val
      },
       set: function defineSet(newVal) {
        console.log(`set key: ${key} val: ${newVal}`)
        // 还记得我们上面讨论的闭包么
        // 此处将新的值赋给val，保存在内存中，从而达到赋值的效果
        val = newVal
      }
    })
  }
  function observe(data) {
    Object.keys(data).forEach(function(key) {
      defineReactive(data, key, data[key])
    })
  }
  
  let test = [1, 2, 3]
  // 初始化
  observe(test)
  ```

  打印的过程可以解释为：
  
  - 找到`test`变量指向的内存位置为一个数组，长度为3并打印，但并不知道索引对应的值是多少
  - 便利索引
  
  接下来我们做如下操作
  
  ![](https://pic.superbed.cn/item/5cbfbe543a213b0417c3219a)

- `push`时，新增了索引并且改变了长度，但新索引未被`observe`
- 修改新的索引对应的值
- 弹出新的索引对应的值
- 弹出索引被`observe`的值时触发了`get`
- 此时再去给原索引赋值时发现并没有触发被`observe`的`set`，由此可见数组索引被删除后就不会被`observe`到了。

那对象的属性被删除后是否还可以被`observe`到么？

![](https://pic.superbed.cn/item/5cbfc59d3a213b0417c38ff5)

- 修改索引为1的值，出发了`set`
- `unshift`时，会将索引为0和1的值遍历出来存放，然后重新赋值

当我们给`length`赋值时，可以看见并不会遍历数组去赋值索引

```javascript
var arr = new Array(1, 2, 3)
arr.length = 5
arr		// [1, 2, 3, empty × 2]
```

### 总结

对于`Object.defineProperty`来说，处理对象和数组一样，只是在初始化时去改写`get`和`set`达到监测数组或对象的变化。对于新增的属性，需要手动再初始化。

对于数组来说，只不过特别了点，某些方法例如`push`、`unshift`等也会新增索引。对于新增的索引亦可以添加`observe`从而达到监听的效果。而`pop`和`shift`则会删除更新索引，也会出发`Object.defineProperty`的`get`和`set`。对于重新赋值`length`的数组，不会新增索引，因为不清楚新增的索引数量。

> 所以在Vue中我们是可以显式的通过调用`vm.$set`监听对象新增的键（`key`）。但这样相对来讲比较损耗性能，所以尤大用了另一种 “奇技淫巧” 来保证数组的更新可以实时同步到`data`中

```javascript
const aryMethods = ['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'];
const arrayAugmentations = [];

aryMethods.forEach((method)=> {

    // 这里是原生Array的原型方法
    let original = Array.prototype[method];

   // 将push, pop等封装好的方法定义在对象arrayAugmentations的属性上
   // 注意：是属性而非原型属性
    arrayAugmentations[method] = function () {
        console.log('我被改变啦!');

        // 调用对应的原生方法并返回结果
        return original.apply(this, arguments);
    };
});

let list = ['a', 'b', 'c'];
// 将我们要监听的数组的原型指针指向上面定义的空数组对象
// 别忘了这个空数组的属性上定义了我们封装好的push等方法
list.__proto__ = arrayAugmentations;
list.push('d');  // 我被改变啦！ 4

// 这里的list2没有被重新定义原型指针，所以就正常输出
let list2 = ['a', 'b', 'c'];
list2.push('d');  // 4
```

这部分在 [《Proxy && defineProperty之实现双向绑定》](https://burning-shadow.github.io/2019/04/12/Proxy-&&-defineProperty%E4%B9%8B%E5%AE%9E%E7%8E%B0%E5%8F%8C%E5%90%91%E7%BB%91%E5%AE%9A/) 一文中有介绍，并由此过渡到了`Proxy`实现双向绑定上，感兴趣的boy可以去看一下~