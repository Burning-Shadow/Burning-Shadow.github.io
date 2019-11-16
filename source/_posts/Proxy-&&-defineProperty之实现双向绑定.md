---
title: Proxy && defineProperty之实现双向绑定
date: 2019-04-12 17:06:00
categories: Vue
tags:
 - Proxy
 - defineProperty
 - 双向绑定
---

**Vue三要素**：

- 响应式：如何监听数据变化（双向绑定）
- 模板引擎：如何解析模板
- 渲染：`Vue`如何将监听到的数据变化和解析后的 HTML 进行渲染

但凡涉及到 MVVM 框架就不得不提到双向绑定原理，也就是数据劫持。而在此之前我们已经尝试着解决过这个问题（详情请看[《Vue双向绑定原理及实现》](https://burning-shadow.github.io/2019/03/31/Vue%E5%8F%8C%E5%90%91%E7%BB%91%E5%AE%9A%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)）。而对于`Vue3.0`，尤雨溪说过要用 ES6 中新推出的`Proxy`来代替`Object.defineProperty`实现数据劫持。那么我们就一起来看一下为什么作者会做出如此改动吧~

<!--more-->

在此之前需要对`Proxy`有一定的了解。详情请看 [《ES6中的代理模式》](https://burning-shadow.github.io/2019/04/12/ES6%E4%B8%AD%E7%9A%84%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F/#more)



**一定要看上边的两篇文章之后再回来看这篇啊！！！！**



### 数据劫持

在《Vue双向绑定原理及实现》一文中我们介绍到了利用`Object.defineProperty`劫持对象访问器（`set`、`get`），以便于属性发生变化时直接进行下一步操作，以此实现一个真正的`Observer`。

然而，我们有没有注意到我们只是对`Watcher`的`get`、`set`方法进行监听而非全部属性，所以我们用`Object.defineProperty`未免有点杀鸡用牛刀的感觉，而`Proxy`则可以对那13个对象属性进行精确监控监听，解决了您的烦恼，~~实则是居家旅行，杀人越货的好帮手~~



但是，总是千变万化，仍逃不出**发布者-订阅者**模式。我们必须利用代理监听`Watcher`的属性值并对其**劫持**，变化时通知订阅者。

而解析器则负责解析模板中的指令，收集指令所以来的方法和数据，等待数据变化然后渲染模板

最终`Watcher（订阅者）`将`Observer(监听者)`和`Compiler(解析器)`连接起来，并根据`Compiler`所提供的指令进行试图渲染是的数据变化推动视图变化。

图我懒得找了所以就盗了一张，诸位好汉不要介意。

![](<https://ww1.sinaimg.cn/large/007i4MEmgy1g1zx8i92c9j30jr0akmxu.jpg>)



### 基于Object.defineProperty的双向绑定

具体内容我们就不再细讲了，[《Vue双向绑定原理及实现》](https://burning-shadow.github.io/2019/03/31/Vue%E5%8F%8C%E5%90%91%E7%BB%91%E5%AE%9A%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)讲的还是比较清楚的。我们来说说它的缺陷：



我们之前所讲的双向绑定无法监听数组变化。而作者则通过遍历数组中的方法手动解除掉了这个`bug`，原理如下

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

### 基于Proxy的双向绑定

`Proxy`可以直接监听对象相比不需要我多讲了，同样的，`Proxy`也可以监听数组，不必再用上面的那些个奇技淫巧让自己难受了。

也就是说，**`Proxy`可以直接劫持整个对象并返回一个新对象**，不论便利程度还是底层功能都远强于`Object.defineProperty`：

```javascript
const input = document.getElementById('input');
const p = document.getElementById('p');
const obj = {};

const newObj = new Proxy(obj, {
  get: function(target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },
  set: function(target, key, value, receiver) {
    console.log(target, key, value, receiver);
    if (key === 'text') {
      input.value = value;
      p.innerHTML = value;
    }
    return Reflect.set(target, key, value, receiver);
  },
});

input.addEventListener('keyup', function(e) {
  newObj.text = e.target.value;
});
```



而使用过`Vue`的`boys`应该也依稀记着我们通过`$nextTick`来绑定被改变的数组长度之类的问题，而`Proxy`则可以完全避免我们使用那些个让你绞尽脑汁想想想的`$nextTick`，直接改变数组长度，包您满E。

### 其他优势

- 优势就是好用，好用到其他浏览器厂商将会以此为标准持续的优化性能
- `Proxy`返回的是一个新对象，故我们可以只操作新对象达到目的，而不是像`Object.defineProperty`一样笨拙的遍历对象属性并修改



当然，劣势是兼容性问题，而且短时间无法完美适应所有浏览器。这个会随着时间的推移逐渐被抹平的，诸位好汉不必担心。那么，

![告辞](https://ww1.sinaimg.cn/large/007i4MEmgy1g1avrg3n5gj30kq0kqq3j.jpg)