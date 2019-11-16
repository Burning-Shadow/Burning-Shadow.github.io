---
title: 重学Vue（上篇）
date: 2019-05-17 12:13:00
categories: 重学框架系列
tags: 
 - vue
---

入职之前重新刷一遍 vue 文档，算是查漏补缺。格式的话我会按照 vue 官网给出的 API 顺序进行，可能的话会加上自己的一些补充，以备日后复习所用

<!--more-->

### 全局配置 Vue.config

#### Vue.config.errorHandler

```javascript
Vue.config.errorHandler = function (err, vm, info) {
  // handle error
  // info 是 Vue 特定的错误信息，比如错误所在的生命周期钩子
  // 只在 2.2.0+ 可用
}
```

- `2.2.0`后：此钩子也可以捕获组件生命周期钩子里的错误。当此钩子为`undefined`时，被捕获的错误会通过`console.error`输出而避免应用崩溃

- `2.4.0`后：此钩子也会捕获 Vue 自定义事件处理函数内部的错误。

- `2.6.0后：`此钩子也会捕获`v-on` DOM 监听器内部抛出的错误。若任何被覆盖的钩子或处理函数返回一个`Promise`链（如`async`函数），则来自其`Promise`链的错误也会被处理

#### Vue.config.warnHandler

```javascript
Vue.config.warnHandler = function (msg, vm, trace) {
  // trace 是组件的继承关系追踪
}
```

为 Vue 的运行时警告赋予一个自定义处理函数。注意这只会在开发者环境下生效，在生产环境下它会被忽略。

#### Vue.config.keyCodes

```javascript
Vue.config.keyCodes = {
  v: 86,
  f1: 112,
  // camelCase 不可用
  mediaPlayPause: 179,
  // 取而代之的是 kebab-case 且用双引号括起来
  "media-play-pause": 179,
  up: [38, 87]
}
```

这玩意儿用作定义别名。比如上边的那段代码的作用就是为`v-on`自定义键位别名

```html
<input type="text" @keyup.media-play-pause="method">
```

#### 一笔带过

- `Vue.config.slient = true/false`是否取消所有的日志与警告。（鸡肋，没人取消的）
- `Vue.config.optionMergeStrategies`：字定义合并策略的选项，没整懂
- `Vue.config.devtools = true`配置是否允许`vue-devtools`检查代码

### 全局 API

#### Vue.extend( options )

这玩意儿顾名思义，就是继承。使用基础 Vue 构造器，创建一个“子类”。参数是一个包含组件选项的对象。其创建的“子类”是全局的

```html
<div id="mount-point"></div>
<script>
// 创建构造器
var Profile = Vue.extend({
  template: '<p>{{extendData}}</br>实例传入的数据为:{{propsExtend}}</p>',//template对应的标签最外层必须只有一个标签
  data: function () {
    return {
      extendData: '这是extend扩展的数据',
    }
  },
  props:['propsExtend']
// 创建 Profile 实例，并挂载到一个元素上。可以通过proposData传参
new Profile({propsData:{propsExtend:'我是实例传入的数据'}}).$mount('#app-extend')
</script>
```

#### Vue.nextTick( [callback, context] )

- 参数
  - `{Function} [callback]`
  - `{Object} [callback]`

> 在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM

```javascript
<template>
  <div>
    <div ref="text">{{text}}</div>
    <button @click="handleClick">text</button>
  </div>
</template>
export default {
  data () {
    return {
      text: 'start'
    };
  },
  methods () {
    handleClick () {
      this.text = 'end';
      console.log(this.$refs.text.innerText);//打印“start”
    }
  }
}
// 打印结果为 start
// 其进行 DOM 更新前会缓存到一个队列中，一个事件循环周期后统一更新
```

这玩意儿好理解。我们在项目开发中应该也曾使用过这东西。当我们在项目中的`beforeCreate`和`created`发送 ajax 到后台，数据返回时发现 DOM 节点还未生成无法操作节点时，此时我们就会用到`this.$nextTick`

其内部机制是将处理 DOM 更新的操作都缓存到一个队列中，在一个事件循环结束之后刷新队列，统一执行 DOM 更新操作。

通常情况下我们不需要关心这个问题，而如果想在 DOM 状态更新后做点什么则需要用到`nextTick`

由于 vue 生命周期的`created`钩子函数进行的 DOM 操作要放在`Vue.nextTick()`的回调函数中。因为`created()`钩子函数执行的时候 DOM 操作的 JS 代码放进`Vue.nextTick()`的回调函数中。而与之对应的`mounted`钩子函数，该钩子函数执行时所有的 DOM 挂载已完成，此时该钩子函数进行任何 DOM 操作都不会发生此问题

> 2.1.0 起新增：如果没有提供回调且在支持 Promise 的环境中，则返回一个 Promise。请注意 Vue 不自带 Promise 的 polyfill，所以如果你的目标浏览器不原生支持 Promise (IE：你们都看我干嘛)，你得自己提供 polyfill。

#### Vue.set(target, propertyName/index, value)

- 参数
  - `{Object | Array} target`
  - `{string | number} propertyName/index`
  - `{any} value`

> **我们用其该全局配置方法向响应式对象中添加一个属性，并确保此新属性同样是响应式的，且<u>触发视图更新</u>。它必须用于向响应式对象上添加新属性，因为 Vue 无法探测普通的新增属性（eg: `this.myObject.newProperty = 'hi'`）**

上面是官方文档的介绍，那么我们来说说人话。这玩意儿是干啥的

举个栗子

```javascript
const vueInstance = new Vue({
  data: {
    arr: [1, 2],
    obj1: {
        a: 3
    }
  }
});

vueInstance.$data.arr[0] = 3;  // 这种骚操作页面不会重新渲染
vueInstance.$data.obj1.b = 3;  // 这种骚操作页面不会重新渲染
```

而文档中我们回头看一遍，**该 API 用于向响应式对象中添加一个属性，并确保该属性同样是响应式的，且<u>触发视图更新</u>。**

所以我们可以通过**向响应式对象添加响应式属性**来触发试图更新

```javascript
Vue.set(vueInstance.$data.arr, 0, 3);  // 这样操作数组可以让页面重新渲染
vueInstance.$set(vueInstance.$data.arr, 0, 3); // 这样操作数组也可以让页面重新渲染
Vue.set(vueInstance.$data.obj1, b, 3);  // 这样操作对象可以让页面重新渲染
vueInstance.$set(vueInstance.$data.obj1, b, 3); // 这样操作对象也可以让页面重新渲染
```

#### Vue.delete(target, propertyName/index)

- 参数
  - `{Object | Array} target`
  - `{string | number} propertyName/index`

> 删除对象的属性。如果对象是响应式的，确保删除能触发更新视图。这个方法主要用于避开 Vue 不能检测到属性被删除的限制，但是你应该很少会使用它。

#### Vue.directive(id, [definition])

- 参数
  - `{string} id`
  - `{Function | Object} [definition]`

```javascript
// 注册
Vue.directive('my-directive', {
  bind: function () {},
  inserted: function () {},
  update: function () {},
  componentUpdated: function () {},
  unbind: function () {}
})

// 注册 (指令函数)
Vue.directive('my-directive', function () {
  // 这里将会被 `bind` 和 `update` 调用
})

// getter，返回已注册的指令
var myDirective = Vue.directive('my-directive')
```

用法如上例所示，注册自定义指令。如果有兴趣的话可以看一下[文档](<https://cn.vuejs.org/v2/guide/custom-directive.html>)

#### Vue.filter(id, [definition])

**参数**：

- `{string} id`
- `{Function} [definition]`

这东西大家伙儿应该也有所涉及，定义全局过滤器。我们通过一个例子来解释吧

在之前写项目时用到了`Moment`插件用来截获当前时间并令其以我们想要的格式输出出来，故注册全局过滤器，将双花括号中的部分按照后边的格式输出。过滤器注册如下（在`main.js`中）

```javascript
import Moment from 'moment'
Moment.locale('zh-cn') //设置中文显示

// 定义moment全局日期过滤器
// {{ 'xxx' | convertTime('yyy-mm-dd')}}
// {{ 'xxx' | convertTime('yyy年mm月dd日')}}
Vue.filter('convertTime', (data, formatStr) => {
    return Moment(data).format(formatStr)
})

// 相对时间（多久之前）
Vue.filter('relativeTime', (previousTime) => {
    return Moment(previousTime).fromNow()
})
```

而其使用如下

```javascript
{{msg.add_time | relativeTime }}
{{msg.add_time | convertTime }}
```

#### Vue.component(id, [definition])

**参数**：

- `{string} id`
- `{Function | Object} [definition]`

我们可以用这东西注册或获取全局组件。注册还会自动使用给定的`id`设置组件的名称

```javascript
// 注册组件，传入一个扩展过的构造器
Vue.component('my-component', Vue.extend({ /* ... */ }))

// 注册组件，传入一个选项对象 (自动调用 Vue.extend)
Vue.component('my-component', { /* ... */ })

// 获取注册的组件 (始终返回构造器)
var MyComponent = Vue.component('my-component')
```

这个就没有太多可介绍的了。项目里有的是这样的东西。一般来讲我们会将常用组件放在项目之中以避免其重复引入

#### Vue.use(plugin)

**参数**：

- `{Object | Function} plugin`

我们引入插件之后（`import`）通过此进行安装。最常用的就是我们的`VueRouter`插件的引入（本例中还加了个`iView`）

```javascript
import Vue from 'vue'
import App from './App'
import VueRouter from 'vue-router'
import router from './router'
import iView from 'iview'

Vue.use(VueRouter)
Vue.use(iView)
```

#### Vue.compile(template)

- **参数**：
  - `{string} template`

这个全局方法用于**在`render`函数中编译模板字符串**。只有在独立构建时有效。

```javascript
var res = Vue.compile('<div><span>{{ msg }}</span></div>')

new Vue({
  data: {
    msg: 'hello'
  },
  render: res.render,
  staticRenderFns: res.staticRenderFns
})
```

这个真的没用过，所以就贴上了官方文档的代码。

[《在渲染函数&JSX》](<https://cn.vuejs.org/v2/guide/render-function.html>)文档中讲述了一个在节点中插入数组的例子。我们若通过模板则冗长而可维护性差，但若直接通过渲染函数直接编译例子中的代码则简单而明了。之后如果需要用到这里再进行补充吧。

#### Vue.observable(object)

- **参数**：
  - `{Object} object`

- **用法**：

让一个对象可响应。Vue 内部会用它来处理 `data` 函数返回的对象。

返回的对象可以直接用于[渲染函数](https://cn.vuejs.org/v2/guide/render-function.html)和[计算属性](https://cn.vuejs.org/v2/guide/computed.html)内，并且会在发生改变时触发相应的更新。也可以作为最小化的跨组件状态存储器，用于简单的场景：

```javascript
const state = Vue.observable({ count: 0 })

const Demo = {
  render(h) {
    return h('button', {
      on: { click: () => { state.count++ }}
    }, `count is: ${state.count}`)
  }
}
```

#### Vue.version

这个的话就是对版本的判断了。不过咱们一般人都用不着

```javascript
var version = Number(Vue.version.split('.')[0])

if (version === 2) {
  // Vue v2.x.x
} else if (version === 1) {
  // Vue v1.x.x
} else {
  // Unsupported versions of Vue
}
```

### 选项 & 数据

#### data

- **类型**：`Object | Function`

- **限制**：组件的定义只接受 `function`。

- **详细**：

  Vue 实例的数据对象。Vue 将会递归将 data 的属性转换为 getter/setter，从而让 data 的属性能够响应数据变化。**对象必须是纯粹的对象 (含有零个或多个的 key/value 对)**：浏览器 API 创建的原生对象，原型上的属性会被忽略。大概来说，data 应该只能是数据 - 不推荐观察拥有状态行为的对象。

  一旦观察过，不需要再次在数据对象上添加响应式属性。因此推荐在创建实例之前，就声明所有的根级响应式属性。

  实例创建之后，可以通过 `vm.$data` 访问原始数据对象。Vue 实例也代理了 data 对象上所有的属性，因此访问 `vm.a` 等价于访问 `vm.$data.a`。

  以 `_` 或 `$` 开头的属性 **不会** 被 Vue 实例代理，因为它们可能和 Vue 内置的属性、API 方法冲突。你可以使用例如 `vm.$data._property` 的方式访问这些属性。

  当一个**组件**被定义，`data` 必须声明为返回一个初始数据对象的函数，因为组件可能被用来创建多个实例。如果 `data` 仍然是一个纯粹的对象，则所有的实例将**共享引用**同一个数据对象！通过提供 `data` 函数，每次创建一个新实例后，我们能够调用 `data` 函数，从而返回初始数据的一个全新副本数据对象。

  如果需要，可以通过将 `vm.$data` 传入 `JSON.parse(JSON.stringify(...))` 得到深拷贝的原始数据对象。

#### props

这玩意儿就是用来接收从父组件中传递而来的数据。格式的话可以是基本类型亦可以是引用类型。

- `type`: 可以是下列原生构造函数中的一种：`String`、`Number`、`Boolean`、`Array`、`Object`、`Date`、`Function`、`Symbol`、任何自定义构造函数、或上述内容组成的数组。会检查一个 prop 是否是给定的类型，否则抛出警告。Prop 类型的[更多信息在此](https://cn.vuejs.org/v2/guide/components-props.html#Prop-类型)。
- `default`: `any`
  为该 prop 指定一个默认值。如果该 prop 没有被传入，则换做用这个值。对象或数组的默认值必须从一个工厂函数返回。
- `required`: `Boolean`
  定义该 prop 是否是必填项。在非生产环境中，如果这个值为 truthy 且该 prop 没有被传入的，则一个控制台警告将会被抛出。
- `validator`: `Function`
  自定义验证函数会将该 prop 的值作为唯一的参数代入。在非生产环境下，如果该函数返回一个 falsy 的值 (也就是验证失败)，一个控制台警告将会被抛出。你可以在[这里](https://cn.vuejs.org/v2/guide/components-props.html#Prop-验证)查阅更多 prop 验证的相关信息。

#### computed

计算属性会被混入到 Vue 实例中。所有`getter`和`setter`的`this`上下文自动绑定为 Vue 实例

其结果会被缓存，**除非依赖的响应式属性变化才会重新计算**。若某个依赖为非响应式属性或在该实例范畴之外，则计算属性不会被更新

#### methods

用以存储方法，不应使用箭头函数来定义`method`函数（否则你自己调用就是个问题）

#### watch

- **类型**：`{ [key: string]: string | Function | Object | Array }`

- **详细**：

  一个对象，键是需要观察的表达式，值是对应回调函数。值也可以是方法名，或者包含选项的对象。**Vue 实例将会在实例化时调用 `$watch()`，遍历 watch 对象的每一个属性**。

### DOM

#### el

- **类型**：`string | Element`

- **限制**：只在由 `new` 创建的实例中遵守。

- **详细**：

  提供一个在页面上已存在的 DOM 元素作为 Vue 实例的挂载目标。可以是 CSS 选择器，也可以是一个 HTMLElement 实例。

  在实例挂载之后，元素可以用 `vm.$el` 访问。

  如果在实例化时存在这个选项，实例将立即进入编译过程，否则，需要显式调用 `vm.$mount()` 手动开启编译。

> 官方文档上总结的这些个东西说白了就是被挂载对象。这玩意儿咱们一般在开头创建、访问的话，就我而言仅仅是在控制台里打印过那么一两次瞅瞅上边挂载了那么多的属性（具体是啥我承认我并没有看），其余的用处我也说不好。
>
> 由于挂载点挂载的元素会被 Vue 替换为 DOM 元素，所以正常脑回路的`boy`不会把`<html>`或者`<body>`当作挂载点。

>**如果 `render` 函数和 `template` 属性都不存在，挂载 DOM 元素的 HTML 会被提取出来用作模板，此时，必须使用 Runtime + Compiler 构建的 Vue 库。**（这个是抄文档的，我自己没理解也没用过这俩工具）

#### template

- **类型**：`string`

- **详细**：

  一个字符串模板作为 Vue 实例的标识使用。模板将会 **替换** 挂载的元素。挂载元素的内容都将被忽略，除非模板的内容有分发插槽。

#### render

- **类型**：`(createElement: () => VNode) => VNode`

- **详细**：

  字符串模板的代替方案，允许你发挥 JavaScript 最大的编程能力。该渲染函数接收一个 `createElement` 方法作为第一个参数用来创建 `VNode`。

  如果组件是一个函数组件，渲染函数还会接收一个额外的 `context` 参数，为没有实例的函数组件提供上下文信息。

这东西就是咱们之前提过的渲染函数。由于 Vue 会帮助你渲染，而有时候恰巧手动渲染更加符合开发者的需求，此时就可以使用`render`函数。个人感觉，这玩意儿决定了一个前端开发者的上限。

在此再次放一遍当时的链接：[《渲染函数&JSX》](<https://cn.vuejs.org/v2/guide/render-function.html>)

> Vue 选项中的 `render` 函数若存在，则 Vue 构造函数不会从 `template` 选项或通过 `el` 选项指定的挂载元素中提取出的 HTML 模板编译渲染函数。

#### renderError

- **类型**：`(createElement: () => VNode, error: Error) => VNode`

- **详细**：

  **只在开发者环境下工作。**

  当 `render` 函数遭遇错误时，提供另外一种渲染输出。其错误将会作为第二个参数传递到 `renderError`。这个功能配合 hot-reload 非常实用。

- **示例**：

  ```javascript
  new Vue({
    render (h) {
      throw new Error('oops')
    },
    renderError (h, err) {
      return h('pre', { style: { color: 'red' }}, err.stack)
    }
  }).$mount('#app')
  ```

（这个没用过所以就照抄文档了）

