---
title: 帮你干苦活儿累活儿的Vue.mixin
categories: Vue
tags:
 - vue
 - Vue.mixin()
---

​	最近产品老哥给我派了个活儿：埋点。这可是个苦活儿而且还很重要容不得差池。

其中很重要的一个参数就是附带上相应组件开始渲染和渲染结束的时间，以此来作为统计表项的一部分。很简单嘛不就是在 `created` 整一个 `new Date().getTime()` 再在 `mounted` 里也写一个，把值存储在 随便一个 `localStorage` 或者 `sessionStorage` 里就行了呗~

​	就这么干！

​	不过好累呀！！！只此一项就已经让人筋疲力竭了好吗再加上其他的一些可变参数配置真的是要了我的命。

​	这时候我们的希望之光 `Vue.mixin()` 就出场啦~

<!--more-->

### 从一个过滤器开始

我们在项目中也许需要将要涉及到过滤数据。每一个组件之中都定义一个 `filterMsg` 显然不太现实。造成代码冗余的同时也不利于维护。那么怎么办呢？

我们直接在 `App.vue` 或者是 `main.js` 里引入我们的主角儿 `Vue.mixin()` 就可以咯~

项目实战中我就遇到过一个需要对字符串进行过滤截取的需求。

```javascript
Vue.mixin({
  filters: {
    $_filterString: (val) = > {
      return val.length > 100? val.slice(0, 100): val;
    }
  }
})
```

这里强调一下我们传入的是一整个对象哦~

然后我们就可以在 `.vue` 里直接调用咯

```html
<template>
  <div>
    {{ string | $_filterString }} 
  </div>
</template>
```

这时候小伙伴也许会发现，你干的这活儿其实定义一个全局的过滤器不也能干么？

### 有些事儿是全局过滤器干不了的

刚才咱们是操纵 `v-text` 的，那如果是让你操纵 HTML 元素呢？

eg：通过状态码来展示不同的图标（此处的图标我们就默认使用 `elementUI` 的 `icon` 了啊）

一般人思路是

```html
...
<span v-if="item.iconType === 1" class="icon icon-up"></span>
<span v-if="item.iconType === 2" class="icon icon-down"></span>
<span v-if="item.iconType === 3" class="icon icon-left"></span>
<span v-if="item.iconType === 4" class="icon icon-right"></span>
...
```

但是emmmmm

我们换一种思路吧还是。同样是过滤器功能的 `mixin` 我们能不能把他用在 `v-html` 中呢？

先在 `mixin` 文件夹里定义一个 `config.js`

```javascript
export const iconStatus = {
  1: "<span class='icon icon-up'></span>",
  2: "<span class='icon icon-down'></span>",
  3: "<span class='icon icon-left'></span>",
  4: "<span class='icon icon-right'></span>"
}
```

然后再在 `filter.js` 里引入

```javascript
import { iconStatus } from "./config"
export default {
  $_filterIcon: (value) => {
      return iconStatus[value] || "icon undefined"
  }
}
```

渲染的时候我们就要用 `v-html` 来渲染啦

```html
<span v-html="$options.filters.$_filterIcon(item.iconType)"></span>
```

### 但是咱们有时候用不着全局的呀

刚才说了咱最近的工作就是埋点，所以这种因地制宜的办法显然更适合咱。

很多时候比如我们在一部分的 `created` 里需要打印其时间戳，而另一部分则不需要。此时就需要我们在全局定义 `Vue.mixin({created: ()=>{ console.log(new Date().getTime()) }})` ，让每一个 `.vue` 文件都打印一个时间戳好像就不太合适了。那么怎么因地制宜的进行对 `mixin` 所定义的函数的引入呢？

官方文档给出了我们答案——[混入](https://cn.vuejs.org/v2/guide/mixins.html#)

> **选项合并**：
>
> 当组件和混入对象含有同名选项时，这些选项将以恰当的方式进行“合并”。
>
> 比如，**数据对象在内部会进行递归合并**，并在**发生冲突时以组件数据优先**。

初始例子在官方文档里有，在此就不再赘述，我们说一些和项目有关的。

初始的混入其实就可以满足我们的需求。在 `.vue` 文件中只需要在 `mixins` 对应数组中放入我们所配置的对象即可将其并入到该 `.vue` 组件的生命周期之中。

还拿刚才我们所说的时间戳来说。在向后台发送的数据之中我们需要获取两个时间点：**进入页面时间**和**动作发生时间**。

我们需要一个 `mixinConfig.js`：

```javascript
export default {
  mounted() {
    localStorage.setItem("startTime", new Date().getTime())
  },
  methods: {
    getTime() {
      localStorage.setItem("triggerTime", new Date().getTime());
    }
  }
}
```

在项目之中引入并在合适的时间调用

```javascript
import mixinConfig from "../utils/mixinConfig.js"

export default {
  name: "aaa"
  data() {},
  mixins: [mixinConfig],
  methods: {
    clickBtn() {
      this.getTime();	// 调用 mixinConfig 中的时间戳
      this.$axios.get("xxxxx", axiosConfig)
    },
    ....
  },
  created() {},
  ....
}
```

