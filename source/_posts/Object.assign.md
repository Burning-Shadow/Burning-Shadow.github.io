---
title: Object.assign
categories: ES6
tags:
 - Object.assign()
 - ES6
---

这东西各位有没有觉得有点眼熟？不错，正是深浅拷贝时候用到的知识。

而本章我们则着手于介绍一下浅拷贝`Object.assign`的实现原理，然后带你手动实现一个浅拷贝。

<!--more-->

浅拷贝，顾名思义，浅尝则止。

将所有**可枚举属性**的值从一个或多个源对象复制到目标对象，同时返回目标对象。

```javascript
Object.assign(target, ...sources)
```

其中 `target` 是目标对象，`sources` 是源对象，可以有多个，返回修改后的目标对象 `target`。

如果目标对象中的属性具有相同的键，则属性将被源对象中的属性覆盖。后来的源对象的属性将类似地覆盖早先的属性。

### 浅拷贝特点

- 拷贝第一层的基本类型值
- 拷贝第一层的引用类型**地址**
- `String`、`Symbol`类型的属性都会被拷贝
- 不会跳过那些值为`null`或`undefined`的源对象

### 实现

````javascript
function shallowCopy(source, target = {}) {
    var key;
    for (key in source) {
        if (source.hasOwnProperty(key)) {
            target[key] = source[key];
        }
    }
    return target;
}
````

通过`hasOwnProperty()`可以防止向上继续拷贝。只对本层处理。