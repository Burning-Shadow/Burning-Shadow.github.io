---
title: Map && Set
date: 2019-04-13 00:42:00
categories: ES6
tags:
 - Map
 - Set
 - ES6
---

ES6 提供了新的数据结构 Map && Set，类似于数组但又有些区别，我们一起来看一下

<!--more-->

### Set

- `Set`结构不会添加重复值，可以用作数组去重

- 但是空对象不尽相等，所以这段代码是可以完成的

  - ```javascript
    let set = new Set();
    
    set.add({});
    set.size // 1
    
    set.add({});
    set.size // 2
    ```

- 向`Set`添值的时候不会发生类型转换，所以字符串和数字是不同的值

#### Set的属性和方法

##### 属性

| 属性                        | 作用                        |
| --------------------------- | --------------------------- |
| `Set.prototype.size`        | 返回`Set`实例的成员总数     |
| `Set.prototype.constructor` | 构造函数，默认就是`Set`函数 |

##### 方法

| 方法            | 作用                                         |
| --------------- | -------------------------------------------- |
| `add(value)`    | 添加某个值，返回 Set 结构本身                |
| `delete(value)` | 删除某个值，返回一个布尔值，表示删除是否成功 |
| `has(value)`    | 返回一个布尔值，表示该值是否为`Set`的成员    |
| `clear()`       | 清除所有成员，没有返回值                     |
| `Array.from()`  | 将 Set 结构转为数组                          |

##### 遍历操作

| 方法        | 作用                     |
| ----------- | ------------------------ |
| `keys()`    | 返回键名的遍历器         |
| `values()`  | 返回键值的遍历器         |
| `entries()` | 返回键值对的遍历器       |
| `forEach()` | 使用回调函数遍历每个成员 |

> 由于`Set`结构没有键名，只有键值（或者说键名键值是同一个值），所以`keys()`和`values()`方法对于他来说完全一致

#### 用法

##### 并集，交集和差集

```javascript
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// 并集
let union = new Set([...a, ...b]);
// Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...a].filter(x => b.has(x)));
// set {2, 3}

// 差集
let difference = new Set([...a].filter(x => !b.has(x)));
// Set {1}
```

#### WeakSet

这东西应该算是`Set`的一个子集吧，但是与`Set`有两点不同

- 成员只能是对象
- 对对象都是弱引用，故垃圾回收机制不考虑`WeakSet`对该对象的引用。

##### 方法

| 方法                              | 作用                                                |
| --------------------------------- | --------------------------------------------------- |
| `WeakSet.prototype.add(value)`    | 向 WeakSet 实例添加一个新成员                       |
| `WeakSet.prototype.delete(value)` | 清除 WeakSet 实例的指定成员                         |
| `WeakSet.prototype.has(value)`    | 返回一个布尔值，表示某个值是否在 WeakSet 实例之中。 |

### Map

- `Map`也是一个类似于对象的集合，不同的是它可以用任意类型做键名而非传统`Object`一样只能用`String`值作键名
- 任何具有`Iterator`接口、且每个成员都是一个双元素的数组的数据结构都可以当作`Map`的参数
- 同一个键多次赋值，后面的值会覆盖前面的值
- 只要两个键值**完全相等**那么`Map`就会将其视为同一个键

```javascript
const set = new Set([
  ['foo', 1],
  ['bar', 2]
]);
const m1 = new Map(set);
m1.get('foo') // 1

const m2 = new Map([['baz', 3]]);
const m3 = new Map(m2);
m3.get('baz') // 3
```

#### 属性

| 属性              | 作用                                               |
| ----------------- | -------------------------------------------------- |
| `size`            | 返回`Map`结构的成员总数                            |
| `set(key, value)` | 设置`key`对应的值为`value`，并返回整个`Map`结构    |
| `get(key)`        | 读取`key`对应的键值，若找不到则返回`undefined`     |
| `has(key)`        | 返回一个`boolean`，表示某个键是否在当前`Map`对象中 |
| `delete(key)`     | 删除某个键，成功返回`true`，失败返回`false`        |
| `clear()`         | 清除所有成员                                       |

#### 遍历方法

| 方法        | 作用                     |
| ----------- | ------------------------ |
| `keys()`    | 返回键名的遍历器         |
| `values()`  | 返回键值的遍历器         |
| `entries()` | 返回键值对的遍历器       |
| `forEach()` | 使用回调函数遍历每个成员 |

#### 和其他数据结构的相互转换

##### Map转数组

```javascript
const myMap = new Map()
  .set(true, 7)
  .set({foo: 3}, ['abc']);
[...myMap]
// [ [ true, 7 ], [ { foo: 3 }, [ 'abc' ] ] ]
```

##### 数组转Map

```javas
new Map([
  [true, 7],
  [{foo: 3}, ['abc']]
])
// Map {
//   true => 7,
//   Object {foo: 3} => ['abc']
// }
```

##### Map转对象

```javascript
function strMapToObj(strMap) {
  let obj = Object.create(null);
  for (let [k,v] of strMap) {
    obj[k] = v;
  }
  return obj;
}

const myMap = new Map()
  .set('yes', true)
  .set('no', false);
strMapToObj(myMap)
// { yes: true, no: false }
```

##### 对象转Map

```javascript
function objToStrMap(obj) {
  let strMap = new Map();
  for (let k of Object.keys(obj)) {
    strMap.set(k, obj[k]);
  }
  return strMap;
}

objToStrMap({yes: true, no: false})
// Map {"yes" => true, "no" => false}
```

##### Map转JSON

**Map键名都是字符串**：转为对象JSON

```javascript
function strMapToJson(strMap) {
  return JSON.stringify(strMapToObj(strMap));
}

let myMap = new Map().set('yes', true).set('no', false);
strMapToJson(myMap)
// '{"yes":true,"no":false}'
```

**Map键名有非字符串**：转为数组JSON

##### JSON转Map

**键名为字符串**

```javascript
function jsonToStrMap(jsonStr) {
  return objToStrMap(JSON.parse(jsonStr));
}

jsonToStrMap('{"yes": true, "no": false}')
// Map {'yes' => true, 'no' => false}
```

**JSON是一个数组，且每个数组成员本身又是一个有两个成员的数组**

```javascript
function jsonToMap(jsonStr) {
  return new Map(JSON.parse(jsonStr));
}

jsonToMap('[[true,7],[{"foo":3},["abc"]]]')
// Map {true => 7, Object {foo: 3} => ['abc']}
```

#### WeakMap

- `WeakMap`只接受对象作为键名（`null`除外）
- `WeakMap`的键名所指向的对象不计入垃圾回收机制