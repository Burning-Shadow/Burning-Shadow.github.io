---
title: JavaScript中的map和reduce
categories: JavaScript
---



以前看js都是云里雾里的，后来学了一些Java感觉稍微能看懂一些，恰逢又回头看到了以前关于js在有道云上的一些笔记，决定做一个关于map和reduce的分析

<!--more-->

## map

>`array.map(function(currentValue,index,arr), thisValue)`
>
>| 参数           | 描述                                                         |
>| -------------- | ------------------------------------------------------------ |
>| *currentValue* | 必须。当前元素的值                                           |
>| *index*        | 可选。当前元素的索引值                                       |
>| *arr*          | 可选。当前元素属于的数组对象                                 |
>| *thisValue*    | 可选。对象作为该执行回调时使用，传递给函数，用作 "this" 的值。<br/>如果省略了 thisValue，或者传入 null、undefined，那么回调函数的 this 为全局对象。 |

- map一般来说针对数组进行操作。但是进行了一个很好的封装使得读者可以清晰的看到被操作数组，以及对数组内每个元素进行操作的函数。我们先看一个小例子：

```javascript
var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];

/*1*/
arr.map(function{
    return x * x;
}); // [1, 4, 9, 16, 25, 36, 49, 64, 81]

/*2*/
var f = function (x) {
    return x * x;
};
var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
var result = [];
for (var i=0; i<arr.length; i++) {
    result.push(f(arr[i]));
}
```

诺，二者的原理其实并无两样，只不过map将函数进行了抽象化使得读者更加清晰的将结果视为：把f(x)作用在Array的每一个元素并把结果生成一个新的Array


**划重点！！！！！**
**我等初学者可以勉强将map视为一个for循环，而其中的方法（函数）则代表了我们要对数组中每个元素进行处理所用到的方法（函数）**

最后放一个将字符串转化为整数的小方法给大家看一下啦

```javascript
var arr = ['1', '2', '3'];
var r;
r = arr.map(function(x){return parseInt(x);});
```

## reduce

> `array.reduce(function(total, currentValue, currentIndex, arr), initialValue)`
>
> | 参数           | 描述                                     |
> | -------------- | ---------------------------------------- |
> | *total*        | 必需。*初始值*, 或者计算结束后的返回值。 |
> | *currentValue* | 必需。当前元素                           |
> | *currentIndex* | 可选。当前元素的索引                     |
> | *arr*          | 可选。当前元素所属的数组对象。           |
> | *initialValue* | 可选。传递给函数的初始值                 |

不知道在座的各位有没有学过数据结构，记不记得里边的广义表章节。

总之其中有一个广义表的存储方法名为**头尾链表存储结构**。叫的花哨其实没那么难懂。即使将广义表分为头尾两部分头部为值，尾部为指针指向后边的元素。后边的元素继续细分为头尾，以此类推
![这里写图片描述](https://pic.superbed.cn/item/5c93be583a213b0417da4761)


![这里写图片描述](https://pic.superbed.cn/item/5c93be8e3a213b0417da4949)


对没错就这么个玩意儿~为了省事我直接把PPT截了个图，大伙儿凑合着看啊

- **那么这东西跟reduce有啥关系？**
  - reduce规定**必须接受两个参数**。reduce()**把结果继续和序列的下一个元素做累计计算**
  - 效果就这样：**[x1, x2, x3, x4].reduce(f) = f(f(f(x1, x2), x3), x4);**----像不像上面说到的广义表头尾链表存储结构倒过来（当然你得把最终的一个元素改成两个）？

有啥用？先来个简单的。对数组进行求和

```javascript
var arr = [1, 3, 5, 7, 9];
arr.reduce(function (x, y) {
    return x + y;
}); 	// 25
```
看到没，效果是不是让你们想起了递归操作？没错。我只考虑前两个元素相加，后边的咱先放下不管。如此如此……就成了嘛~

不光如此，reduce还能将数组中的一组数转化为一个大整数

```javascript
var arr = [1, 3, 5, 7, 9];
arr.reduce(function (x, y) {
    return x * 10 + y;
}); 		// 13579
```
- 最后放一个不使用parseInt()利用map和reduce将字符串数组转化为整数的例子
  - Tips：想办法把一个字符串13579先变成Array——[1, 3, 5, 7, 9]，再利用reduce()就可以写出一个把字符串转换为Number的函数。

```javascript
'use strict';
function string2int(s) {

    //split()分割字符串返回的是一个数组；
    var arr = s.split('');
    return arr.map(function(x){
          //js的弱类型转换，‘-’会将字符串转变为数字，
          //x (乘) 1也是一个道理，但这样为何不直接return s (乘) 1呢;
          return x-0;
    }).reduce(function a(x,y){
       return x*10+y;
    });
    
}
```

如果对 JS 中的数组想有更多了解则可以查阅这篇博客：[《Array类型中的操作方法》](https://burning-shadow.github.io/2019/03/27/Array%E7%B1%BB%E5%9E%8B%E4%B8%AD%E7%9A%84%E6%93%8D%E4%BD%9C%E6%96%B9%E6%B3%95/)

