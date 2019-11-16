---
title: Array类型中的操作方法
date: 2019-07-06 10:13:00
categories: JavaScript
tags:
 - JavaScript
 - Array
---

本篇博客仅用于给自己当作一个API文档，已被随时查询

之前的版本整体思路比较乱，因为学习`reactJS`的原因所以重新整理了一下【因为`reactJS`强调尽可能不更改原始数据】。

话不多说我们开始吧。

<!--more-->

上面我们提到` reactJS`需要保留原始数据，以保留原始记录为后续的操作提供根源。所以我们为此将数组中的方法分为两大类：**改变自身的方法** & **不改变自身的方法**

## 按操作前后是否改变区分

### 改变自身的方法

- `array.copyWithin（target, start [, end = this.length]）`【有兼容性问题】
  用于在数组内的替换操作，即替换元素和被替换元素都是数组内的元素
  参数皆为整数，允许start，end为负数（倒数第n个）

- `array.fill(value [,statrt = 0[, end = this.length]])`
  将数组中指定区间的所有元素的值，都替换成value
  start，end允许为负值，同上

- `array.pop()`
  删除一个数组中的最后一个元素，并且返回这个元素

- `array.push(element1, ...elementN)`
  添加一个或多个元素到数组的末尾，并返回数组新的长度

- `array.reverse()`
  前后颠倒数组中元素的位置，第一个元素会成为最后一个

- `array.shift()`
  删除数组的第一个元素，并返回这个元素

- `array.unshift(element1, ...elementN)`
  在数组的开头插入一个或多个元素，并返回数组的新长度

- `array.sort([function(a, b)])`
  对数组的元素做原地的排序，并返回这个数组。sort可能*不稳定*，默认按照字符串的`unicode`码位点排序

  记a和b是两个将要被比较的元素：

  - 如果函数`function(a, b)`返回值小于0， 则a会排在b之前
  - 如何函数返回值等于0， 则a和b的相对位置不变（并不被保证）
  - 如果函数返回值大于0，则a会排在b之后
  - 比较函数输出结果必须稳定，否则排序的结果将是不确定的

- `array.splice(start, deleteCount[, item1[, item2...])`
  在任意的位置给数组添加或删除任意个元素（拼接），返回被删除的元素组成的数组，没有则返回空数组

  - `start`：开始操作的索引
  - `deleteCount`：要移除的数组元素的个数
  - `itemN`：要添加进数组的元素，如果不指定，则`splice`只删除数组元素

### 不改变自身的方法

- `array.concat(value1, value2.....)`
  将传入的数组或非数组值与原数组合并，组成一个新的数组并返回
- `array.includes(searchElement, [, fromIndex])`[实验性质，es7，可能会改变或删除]
  用来判断当前数组是否包含某指定的值，如果是，则返回`true`，否则`false`
- `array.join([separator = ','])`
  将数组中的所有元素连接成一个字符串(默认用逗号作为分隔符，如果separator是一个空字符串，那么数组中的所有元素将被直接连接)
  如果元素是undefined或者null，则会转化成空字符串
- `array.slice([begin = 0 [, end = this.length - 1]])`
  把数组中一部分的浅复制（`shallow copy`）存入一个新的数组对象中，并返回这个新的数组
- `array.toLocaleString()`
  返回一个字符串表示数组中的元素。数组中的元素将使用各自的`toLocaleString`方法转化成字符串，这些字符串将使用一个特定语言环境的字符串（例如逗号）隔开
- `array.indexOf(searchElement[, fromIndex = 0])`
  返回指定元素能在数组中找到的第一个索引值，否则返回-1
  `fromIndex`可以为负，表示从倒数第n个开始（**此时仍然从前向后查询数组**）
  使用“严格相等”（===）进行匹配
- `array.lastIndexOf(searchElement[, fromIndex = arr.length - 1])`
  返回指定元素在数组中的最后一个的索引，如果不存在则返回-1， **从数组的后面向前查找**

## 遍历方法

- `array.forEach((val, index, arr) => {})`
  让数组的每一项都执行一次给定的函数
  v表示当前项的值，i表示当前索引，a表示数组本身
  *`forEach`遍历的范围在第一次调用 callback前就会确定。调用`forEach`后添加到数组中的项不会被 `callback`访问到。如果已经存在的值被改变，则传递给 `callback`的值是` forEach`遍历到他们那一刻的值。已删除的项不会被遍历到。*
- `array.entries()`
  返回一个Array Iterator对象，该对象包含数组中每一个索引的键值对
- `array.every(callback(val, index, arr){})`
  `callback`只会为那些已经被赋值的索引调用，不会为那些被删除或从来没有被赋值的索引调用（返回的是一个`bollean`值）
- `array.some()`：对每个元素的callback函数结果作逻辑“||”操作
- `array.filter((val, index, arr) => {})`
  使用指定的函数测试所有元素，并创建一个包含所有测试通过的元素的新数组
  callback函数返回一个布尔值，true即通过测试
  callback只会在已经赋值的索引上被调用，对于那些已经被删除或者从未被赋值的索引不会被调用
  *不会改变原数组*
- `array.find((val, index, arr) =>{})`【有兼容性问题目前】
  返回数组中满足测试条件的**第一个元素**，如果没有满足条件的元素，则返回undefined
- `array.keys()`
  返回一个数组索引的迭代器（类似于`array.entries()`方法）
- `array.map((val, index, arr) => {})`
  返回一个由原数组中的每个元素调用一个指定方法后的返回值组成的新数组
  map 不修改调用它的原数组本身（当然可以在 callback 执行时改变原数组）
- `array.reduce(callback[, initialValue])`
  该方法接收一个函数作为累加器（`accumulator`），数组中的每个值（从左到右）开始合并，最终为一个值
  - `previousValue`:上一次调用回调返回的值，或者是提供的初始值（`initialValue`）
  - `currentValue`: 数组中当前被处理的元素
  - `index`： index
  - `array`： 调用的数组

## 按功能区分

### 检测数组

- `Array.isArray(obj)`：确定`obj`是否为数组
- `Object.prototype.toString.call(obj)`

### 转换方法

- `toString()`：返回由数组中每个值的字符串形式拼接而成的，以逗号分隔的字符串
- `valueOf()`：返回 Array 对象的原始值（返回最适合该对象类型的原始值）

### 栈 & 队列

- `push`：入尾（任意个参数）
- `pop`：出尾（1个参数）
- `shift`：出头（1个参数）
- `unshift`：入头（任意个参数）

### 重排序方法

- `reverse()`：反转数组

- ```javascript
  var arr = [0, 1, 10, 15, 5]
  function compare(val1, val2){
      return (val2 - val1)
  }
  arr.sort(compare)
  // [0, 1, 5, 10, 15]
  ```

### 操作方法

- `concat( arr | val )`：连接。参数可为值或数组，直接更新操作对象
- `splice()`：**原数组改变**

  - 删除：`arr.splice(startIndex, delNum)`

  - 插入：`arr.splice(startIndex, 0, insertItem)`

  - 替换：`arr.splice(startIndex, delNum, insertItem)`

    - ```javascript
      var x = [14, 3, 77]
      var y = x.splice(1, 2)
      console.log(x)   // [14]
      console.log(y)   // [3, 77]
      ```
- `slice()`：新建并**返回一个**截取原数组的**新数组**。**原数组不会改变**

  - `arr.slice(startIndex, endIndex)`：返回`arr`的`startIndex ~ endIndex-1`项
  - `arr.slice(startIndex)`：返回`arr`的`startIndex ~ 尾`项
- `indexOf && lastIndexOf`：参数既可为`index`亦可为`value`。若返回“-1”则表示未找到

### 迭代方法

- `every(func)`：全真则真

- `some(func)`：一真则真

- `filter(func)`：返回`true`项组成的数组

- `forEach(func)`：对数组每一项执行函数

- `map(func)`：对数组每一项执行函数，并返回调用结果组成的数组

  - ```javascript
    var arr = [1, 2, 3]
    var mapResult = arr.map((item, index, array)=>{
    	return item*2
    })
    arr			// [1, 2, 3]
    mapResult	// [2, 4, 6]
    ```

### 归并方法

- `reduce && reduceRight`：迭代数组所有项并构建一个最终返回的值

  - `reduce(func [,initialVal])`：从数组的第一项开始逐个遍历到最后

  - `reduceRight(func [,initialVal)`：从数组的最后一项开始，向前遍历到第一项

  - ```javascript
    // 求数组中所有值之和
    var arr = [1, 2, 3]
    var sum = arr.reduce((prev, cur, index, array)=>{
        console.log(prev, cur, index)
        return prev + cur
    })
    console.log(sum)	// 6
    ```


### 数组排序

#### 冒泡排序

```javascript
var arr = [1, 9, 4, 50, 49, 6, 3, 2];
function test(arr){
  if (arr.length <= 1) return arr;//如果数组只有一位，就没有必要比较了
  var index = Math.floor(arr.length / 2);//获取中间值的索引
  var cur = arr.splice(index, 1);//截取中间值，如果此处使用cur=arr[index]; 那么将会出现无限递归的错误
  var left = [], right = [];//小于中间值的放在left数组里，大于的放在right数组
  for (var i = 0; i < arr.length; i++){
    if (cur > arr[i]){
      left.push(arr[i]);
    } else{
      right.push(arr[i]);
    }
  }
  return test(left).concat(cur, test(right));//通过递归，上一轮比较好的数组合并，并且再次进行比较
}
test(arr);
```

#### sort()

```javascript
var arr = [1, 9, 4, 50, 49, 6, 3, 2];
function test(){
  return arr.sort(sortNumber);
}
function sortNumber(a, b){
  return a - b;
}
test();
```

### 数组去重

#### 方法1

```javascript
var arr = [1, 'a', 'a', 'b', 'd', 'e', 'e', 1, 0]
function test(){
  for (var i = 0; i < arr.length; i++){
    for(var j = i + 1; j < arr.length; j++){
      if(arr[i] === arr[j]) arr.splice(j,1);//如果前一个值与后一个值相等，那么就去掉后一个值，splice()可以修改原数组
    }
  }
  return arr;
}
test();
```

#### 方法2

```javascript
var arr = [1, 1, 4, 50, 50, 6, 2, 2];
function test(){
  return arr.filter(function(item,index,array){
    return array.indexOf(item) === index; 
    //或者这样写return array.indexOf(item, index+1) === -1; 如果没有重复项，返回true
    //用filter方法，返回ietm对应的indexOf索引值与本身index索引值相等的值，也就是去掉重复的值，filter本身不修改数组，只是会自动遍历数组，去掉重复值后，那么arr就剩下不重复的了
  });
}
test();//输出Array [ 1, 4, 50, 6, 2 ]
```

#### ES6

```javascript
var arr = [1, 1, 4, 50, 50, 6, 2, 2];
function unique(arr){
  return Array.from(new Set(arr));
}
unique(arr);
```

## 数组降维

```javascript
let arr = [[1],[9, [6, 8, [5, [10]]]]]
function flatterMap(arr) {
    for (let i = 0; i < arr.length; i++) {
        if (Array.isArray(arr[i])) {
            let tempArr = arr[i]
            arr.splice(i, 1, ...tempArr)
            i--
        }
    }
 	return arr
}

flatterMap(arr)  //[1,9,6,8,5,10]
```

摒弃了递归的做法，带来了性能上极大的提升。
