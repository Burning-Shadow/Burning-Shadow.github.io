---
title: 重学Vue（中篇）
categories: 重学框架系列
tags: 
 - Vue
---

上篇我们对全局对象及相关的配置 API 进行了一遍刷文档。那么这次我们从生命周期开始继续刷

<!--more-->

### Vue生命周期钩子

这玩意儿大家伙儿面试的时候应该没少被问。而具体的内容实现的话网上也有不少相关的文档。话不多说先贴一张最经典的图

![](https://pic.superbed.cn/item/5cdeca03697df1fd0ce8c633)

- beforeCreate
  
  - 在实例初始化之后，数据观测 (`data observer`) 和 `event/watcher` 事件配置之前被调用。
  
- created
  
  - 在实例创建完成后被立即调用。在这一步，实例已完成以下的配置：数据观测 (data observer)，属性和方法的运算，`watch/event`事件回调。然而，挂载阶段还没开始，`$el` 属性目前不可见。
  
- beforeMount
  
  - 在挂载开始之前被调用：相关的 `render` 函数首次被调用。（**该钩子在服务器端渲染期间不被调用。**）
  
- mounted
  - `el` 被新创建的 `vm.$el` 替换，并挂载到实例上去之后调用该钩子。如果`root`实例挂载了一个文档内元素，当 `mounted` 被调用时 `vm.$el` 也在文档内。

  - 注意 `mounted` **不会**承诺所有的子组件也都一起被挂载。如果你希望等到整个视图都渲染完毕，可以用 [vm.$nextTick](https://cn.vuejs.org/v2/api/#vm-nextTick) 替换掉 `mounted`：

  ```javascript
  mounted: function () {
    this.$nextTick(function () {
      // Code that will run only after the
      // entire view has been rendered
    })
  }
  ```

  **该钩子在服务器端渲染期间不被调用。**

- beforeUpdate
  
  - 数据更新时调用，发生在虚拟 DOM 打补丁之前。这里适合在更新之前访问现有的 DOM，比如手动移除已添加的事件监听器。（**该钩子在服务器端渲染期间不被调用，因为只有初次渲染会在服务端进行。**）
  
- updated
  - 由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。
  - 当这个钩子被调用时，组件 DOM 已经更新，所以你现在可以执行依赖于 DOM 的操作。然而在大多数情况下，你应该避免在此期间更改状态。如果要相应状态改变，通常最好使用[计算属性](https://cn.vuejs.org/v2/api/#computed)或 [watcher](https://cn.vuejs.org/v2/api/#watch) 取而代之。
  - 注意 `updated` **不会**承诺所有的子组件也都一起被重绘。如果你希望等到整个视图都重绘完毕，可以用 [vm.$nextTick](https://cn.vuejs.org/v2/api/#vm-nextTick) 替换掉 `updated`：
  - 

  ```
  updated: function () {
    this.$nextTick(function () {
      // Code that will run only after the
      // entire view has been re-rendered
    })
  }
  ```

  **该钩子在服务器端渲染期间不被调用。**

- beforeDestory
  
  - 实例销毁之前调用。在这一步，实例仍然完全可用。（**该钩子在服务器端渲染期间不被调用。**）
  
- destoryed
  
  - Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。（**该钩子在服务器端渲染期间不被调用。**）
  
- errorCaptured(2.5.0+)
  - **类型**：`(err: Error, vm: Component, info: string) => ?boolean`
  - **详细**：

  当捕获一个来自子孙组件的错误时被调用。此钩子会收到三个参数：错误对象、发生错误的组件实例以及一个包含错误来源信息的字符串。此钩子可以返回 `false`以阻止该错误继续向上传播。

> 你可以在此钩子中修改组件的状态。因此在模板或渲染函数中设置其它内容的短路条件非常重要，它可以防止当一个错误被捕获时该组件进入一个无限的渲染循环。

- 错误传播规则
  - 默认情况下，如果全局的 `config.errorHandler` 被定义，所有的错误仍会发送它，因此这些错误仍然会向单一的分析服务的地方进行汇报。
  - 如果一个组件的继承或父级从属链路中存在多个 `errorCaptured` 钩子，则它们将会被相同的错误逐个唤起。
  - 如果此 `errorCaptured` 钩子自身抛出了一个错误，则这个新错误和原本被捕获的错误都会发送给全局的 `config.errorHandler`。
  - 一个 `errorCaptured` 钩子能够返回 `false` 以阻止错误继续向上传播。本质上是说“这个错误已经被搞定了且应该被忽略”。它会阻止其它任何会被这个错误唤起的 `errorCaptured` 钩子和全局的 `config.errorHandler`。

#### 代码演示

```javascript
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <script type="text/javascript" src="https://cdn.jsdelivr.net/vue/2.1.3/vue.js"></script>
</head>
<body>

<div id="app">
     <p>{{ message }}</p>
</div>

<script type="text/javascript">
    
  var app = new Vue({
      el: '#app',
      data: {
          message : "xuxiao is boy" 
      },
       beforeCreate: function () {
                console.group('beforeCreate 创建前状态===============》');
               console.log("%c%s", "color:red" , "el     : " + this.$el); //undefined
               console.log("%c%s", "color:red","data   : " + this.$data); //undefined 
               console.log("%c%s", "color:red","message: " + this.message)  
        },
        created: function () {
            console.group('created 创建完毕状态===============》');
            console.log("%c%s", "color:red","el     : " + this.$el); //undefined
               console.log("%c%s", "color:red","data   : " + this.$data); //已被初始化 
               console.log("%c%s", "color:red","message: " + this.message); //已被初始化
        },
        beforeMount: function () {
            console.group('beforeMount 挂载前状态===============》');
            console.log("%c%s", "color:red","el     : " + (this.$el)); //已被初始化
            console.log(this.$el);
               console.log("%c%s", "color:red","data   : " + this.$data); //已被初始化  
               console.log("%c%s", "color:red","message: " + this.message); //已被初始化  
        },
        mounted: function () {
            console.group('mounted 挂载结束状态===============》');
            console.log("%c%s", "color:red","el     : " + this.$el); //已被初始化
            console.log(this.$el);    
               console.log("%c%s", "color:red","data   : " + this.$data); //已被初始化
               console.log("%c%s", "color:red","message: " + this.message); //已被初始化 
        },
        beforeUpdate: function () {
            console.group('beforeUpdate 更新前状态===============》');
            console.log("%c%s", "color:red","el     : " + this.$el);
            console.log(this.$el);   
               console.log("%c%s", "color:red","data   : " + this.$data); 
               console.log("%c%s", "color:red","message: " + this.message); 
        },
        updated: function () {
            console.group('updated 更新完成状态===============》');
            console.log("%c%s", "color:red","el     : " + this.$el);
            console.log(this.$el); 
               console.log("%c%s", "color:red","data   : " + this.$data); 
               console.log("%c%s", "color:red","message: " + this.message); 
        },
        beforeDestroy: function () {
            console.group('beforeDestroy 销毁前状态===============》');
            console.log("%c%s", "color:red","el     : " + this.$el);
            console.log(this.$el);    
               console.log("%c%s", "color:red","data   : " + this.$data); 
               console.log("%c%s", "color:red","message: " + this.message); 
        },
        destroyed: function () {
            console.group('destroyed 销毁完成状态===============》');
            console.log("%c%s", "color:red","el     : " + this.$el);
            console.log(this.$el);  
               console.log("%c%s", "color:red","data   : " + this.$data); 
               console.log("%c%s", "color:red","message: " + this.message)
        }
    })
</script>
</body>
</html>
```

##### create && mounted

我们运行上述代码会发现：

>`beforecreated`：el 和 data 并未初始化 
>`created`:完成了 data 数据的初始化，el没有
>`beforeMount`：完成了 el 和 data 初始化 
>`mounted` ：完成挂载

同时，`beforeMount`时我们发现`el`还是 {{message}} 。这里就是应用的 `Virtual DOM`（虚拟Dom）技术，先把坑占住了。到后面`mounted`挂载的时候再把值渲染进去。

##### update

我们在`chrome`的控制台中输入这样一段代码

```javascript
app.message= 'yes !! I do';
```

我们就可以看到`data`中的值被修改后触发`update`的操作

![](https://pic.superbed.cn/item/5cded899697df1fd0ce95287)

这两个算是比较关键的部分，我们通过控制台打印完成使用。

### 选项&&组合

#### $parent & $children

- **类型**：`Vue instance`

- **详细**：

  指定已创建的实例之父实例，在两者之间建立父子关系。子实例可以用 `this.$parent` 访问父实例，子实例被推入父实例的 `$children` 数组中。

不过这玩意儿权限太大了，慎用！慎用！如果只是组件间通讯那还是老老实实用`props`和`emit吧`

#### mixins

- **类型**：`Array<Object>`

- **详细**：

  `mixins` 选项接受一个混入对象的数组。这些混入实例对象可以像正常的实例对象一样包含选项，他们将在 `Vue.extend()` 里最终选择使用相同的选项合并逻辑合并。举例：如果你的混入包含一个钩子而创建组件本身也有一个，两个函数将被调用。

  Mixin 钩子按照传入顺序依次调用，并在调用组件自身的钩子之前被调用。

- **示例**：

  ```javascript
  var mixin = {
    created: function () { console.log(1) }
  }
  var vm = new Vue({
    created: function () { console.log(2) },
    mixins: [mixin]
  })
  // => 1
  // => 2
  ```

#### extends

- **类型**：`Object | Function`

- **详细**：

  允许声明扩展另一个组件(可以是一个简单的选项对象或构造函数)，而无需使用 `Vue.extend`。这主要是为了便于扩展单文件组件。

  这和 `mixins` 类似。

- **示例**：

  ```javascript
  var CompA = { ... }
  
  // 在没有调用 `Vue.extend` 时候继承 CompA
  var CompB = {
    extends: CompA,
    ...
  }
  ```

#### provide & reject(2.2.0+)

- **类型**：

  - **provide**：`Object | () => Object`
  - **inject**：`Array<string> | { [key: string]: string | Symbol | Object }`

- **详细**：

  `provide` 和 `inject` **主要为高阶插件/组件库提供用例**。并不推荐直接用于应用程序代码中。

  这对选项需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论**组件层次有多深**，并在起上下游关系成立的时间里始终生效。如果你熟悉 React，这与 React 的上下文特性很相似。

  `provide` 选项应该是一个对象或返回一个对象的函数。该对象包含可注入其子孙的属性。在该对象中你可以使用 ES2015 Symbols 作为 key，但是只在原生支持 `Symbol` 和 `Reflect.ownKeys` 的环境下可工作。

  `inject` 选项应该是：

  - 一个字符串数组，或
  - 一个对象，对象的 key 是本地的绑定名，value 是：
    - 在可用的注入内容中搜索用的 key (字符串或 Symbol)，或
    - 一个对象，该对象的：
      - `from` 属性是在可用的注入内容中搜索用的 key (字符串或 Symbol)
      - `default` 属性是降级情况下使用的 value

  > 提示：`provide` 和 `inject` 绑定并不是可响应的。这是刻意为之的。然而，如果你传入了一个可监听的对象，那么其对象的属性还是可响应的。

- **示例**：

  ```javascript
  // 父级组件提供 'foo'
  var Provider = {
    provide: {
      foo: 'bar'
    },
    // ...
  }
  
  // 子组件注入 'foo'
  var Child = {
    inject: ['foo'],
    created () {
      console.log(this.foo) // => "bar"
    }
    // ...
  }
  ```

  利用 ES2015 `Symbols`、函数 `provide` 和对象 `inject`：

  ```javascript
  const s = Symbol()
  
  const Provider = {
    provide () {
      return {
        [s]: 'foo'
      }
    }
  }
  
  const Child = {
    inject: { s },
    // ...
  }
  ```

  > 接下来 2 个例子只工作在 Vue 2.2.1 或更高版本。低于这个版本时，注入的值会在 `props` 和 `data` 初始化之后得到。

  使用一个注入的值作为一个属性的默认值：

  ```javascript
  const Child = {
    inject: ['foo'],
    props: {
      bar: {
        default () {
          return this.foo
        }
      }
    }
  }
  ```

  使用一个注入的值作为数据入口：

  ```javascript
  const Child = {
    inject: ['foo'],
    data () {
      return {
        bar: this.foo
      }
    }
  }
  ```

  > 在 2.5.0+ 的注入可以通过设置默认值使其变成可选项：

  ```javascript
  const Child = {
    inject: {
      foo: { default: 'foo' }
    }
  }
  ```

  如果它需要从一个不同名字的属性注入，则使用 `from` 来表示其源属性：

  ```javascript
  const Child = {
    inject: {
      foo: {
        from: 'bar',
        default: 'foo'
      }
    }
  }
  ```

  与`prop`的默认值类似，你需要对非原始值使用一个工厂方法：

  ```javascript
  const Child = {
    inject: {
      foo: {
        from: 'bar',
        default: () => [1, 2, 3]
      }
    }
  }
  ```

### 选项/其他（属性）

#### name

- **类型**：`string`

- **限制**：只有作为组件选项时起作用。

- **详细**：

  允许组件模板递归地调用自身。注意，组件在全局用 `Vue.component()` 注册时，全局 ID 自动作为组件的 name。

  指定 `name` 选项的另一个好处是**便于调试**。有名字的组件有更友好的警告信息。另外，当在有 [vue-devtools](https://github.com/vuejs/vue-devtools)，未命名组件将显示成 `<AnonymousComponent>`，这很没有语义。**通过提供 `name` 选项，可以获得更有语义信息的组件树。**

  这里提一嘴，之前我们在编写`vue-router`的时候由于复用了组件并且将每一个使用相同组件的`route`都起了一样的名字，所以浏览器给我们报了好多黄色警告，大意是不能重复命名的样子。改成不同的`name`之后就完美的解决了复用组件的问题。

#### delimiters

- **类型**：`Array<string>`

- **默认值**：`["{{", "}}"]`

- **限制**：这个选项只在完整构建版本中的浏览器内编译时可用。

- **详细**：

  改变纯文本插入分隔符。

- **示例**：

  ```javascript
  new Vue({
    delimiters: ['${', '}']
  })
  
  // 分隔符变成了 ES6 模板字符串的风格
  ```

这个貌似没有什么可说的

#### functional

- **类型**：`boolean`

- **详细**：

  使组件无状态 (没有 `data` ) 和无实例 (没有 `this` 上下文)。他们用一个简单的 `render` 函数返回虚拟节点使他们更容易渲染。

这玩意儿说实在的就是函数式组件的一个属性。将其标名为`true`之后则意味它无状态没有[响应式数据](https://cn.vuejs.org/v2/api/#选项-数据))，也没有实例 (没有 `this` 上下文)。

不过一般人应该不这么使

#### model（2.2.0+）

- **类型**：`{ prop?: string, event?: string }`

- **详细**：

  允许一个自定义组件在使用 `v-model` 时定制`prop`和`event`。默认情况下，一个组件上的 `v-model` 会把 `value` 用作 prop 且把 `input` 用作 event，但是一些输入类型比如单选框和复选框按钮可能想使用 `value` prop 来达到不同的目的。使用 `model` 选项可以回避这些情况产生的冲突。

- **Example**：

  ```javascript
  Vue.component('my-checkbox', {
    model: {
      prop: 'checked',
      event: 'change'
    },
    props: {
      // this allows using the `value` prop for a different purpose
      value: String,
      // use `checked` as the prop which take the place of `value`
      checked: {
        type: Number,
        default: 0
      }
    },
    // ...
  })
  ```

  而本质上，`v-model`则是一块绑定了默认事件的语法糖。通过`emit`和`on`来完成对数据的双向绑定
  
  ```html
<my-checkbox v-model="foo" value="some value"></my-checkbox>
  ```

  上述代码相当于：
  
  ```javascript
  <my-checkbox
    :checked="foo"
    @change="val => { foo = val }"
    value="some value">
  </my-checkbox>
  ```

`v-model`就可以理解为一个语法糖。想必诸位好汉应该试过用`$emit`和`$on`实现`v-model`叭。

```html
<!-- 原型式 -->
<input v-model="text" />

<!-- 展开式 -->
<input
  :value="text"
  @input="e => text = e.target.value"
/>
```

### 实例属性

这里的一部分其实已经被大家所熟悉，所以就简短的介绍几个吧。

#### vm.$slots

- 用来访问被[插槽分发](https://cn.vuejs.org/v2/guide/components.html#通过插槽分发内容)的内容。每个[具名插槽](https://cn.vuejs.org/v2/guide/components-slots.html#具名插槽) 有其相应的属性 (例如：`v-slot:foo`中的内容将会在 `vm.$slots.foo` 中被找到)。`default` 属性包括了所有没有被包含在具名插槽中的节点，或 `v-slot:default` 的内容。

  **Note:** `v-slot:foo` is supported in v2.6+. For older versions, you can use the [deprecated syntax](https://cn.vuejs.org/v2/guide/components-slots.html#Deprecated-Syntax).

  在使用[渲染函数](https://cn.vuejs.org/v2/guide/render-function.html)书写一个组件时，访问 `vm.$slots` 最有帮助。

- **示例**：

  ```html
  <blog-post>
    <template v-slot:header>
      <h1>About Me</h1>
    </template>
  
    <p>Here's some page content, which will be included in vm.$slots.default, because it's not inside a named slot.</p>
  
    <template v-slot:footer>
      <p>Copyright 2016 Evan You</p>
    </template>
  
    <p>If I have some content down here, it will also be included in vm.$slots.default.</p>.
  </blog-post>
  ```

  ```javascript
  Vue.component('blog-post', {
    render: function (createElement) {
      var header = this.$slots.header
      var body   = this.$slots.default
      var footer = this.$slots.footer
      return createElement('div', [
        createElement('header', header),
        createElement('main', body),
        createElement('footer', footer)
      ])
    }
  })
  ```

如上例所示，我们可以通过此种方式动态获取插槽中的各属性。我们用的比较少，因为这是渲染函数的写法，今后实力提升了也许会单开一期坑吧

#### vm.$scopedSlots

用来访问[作用域插槽](https://cn.vuejs.org/v2/guide/components-slots.html#作用域插槽)。对于包括 `默认 slot` 在内的每一个插槽，该对象都包含一个返回相应 VNode 的函数。

`vm.$scopedSlots` 在使用[渲染函数](https://cn.vuejs.org/v2/guide/render-function.html)开发一个组件时特别有用。

**注意**：从 2.6.0 开始，这个属性有两个变化：

1. 作用域插槽函数现在保证返回一个 VNode 数组，除非在返回值无效的情况下返回 `undefined`。
2. 所有的 `$slots` 现在都会作为函数暴露在 `$scopedSlots` 中。如果你在使用渲染函数，不论当前插槽是否带有作用域，我们都推荐始终通过 `$scopedSlots`访问它们。这不仅仅使得在未来添加作用域变得简单，也可以让你最终轻松迁移到所有插槽都是函数的 Vue 3。

#### vm.$refs

> 一个对象，持有注册过 [`ref` 特性](https://cn.vuejs.org/v2/api/#ref) 的所有 DOM 元素和组件实例。

这个大家应该比较熟悉。通过在 DOM 节点上绑定属性来获取 DOM 节点。虽然 vue 不推荐咱们直接操作 DOM 但是这玩意儿也得会是不是？

简要说一句，在对应 DOM 节点上绑定属性`ref="aaa"`，然后通过`this.$refs.aaa`就可以获取对应的 节点啦~

#### vm.$listeners

- **类型**：`{ [key: string]: Function | Array<Function> }`

- **只读**
- **解释**

> 包含了父作用域中的 (不含 `.native` 修饰器的) `v-on` 事件监听器。它可以通过 `v-on="$listeners"` 传入内部组件——在创建更高层次的组件时非常有用。

### 实例方法/数据

#### vm.$watch(expOrFn, callback, [options])

- **参数**：

  - `{string | Function} expOrFn`

  - `{Function | Object} callback`

  - ```
    {Object} [options]
    ```

    - `{boolean} deep`
    - `{boolean} immediate`

- **返回值**：`{Function} unwatch`

- **用法**：

  观察 Vue 实例变化的一个表达式或计算属性函数。回调函数得到的参数为新值和旧值。表达式只接受监督的键路径。对于更复杂的表达式，用一个函数取代。

  注意：在变异 (不是替换) 对象或数组时，旧值将与新值相同，因为它们的引用指向同一个对象/数组。**Vue 不会保留变异之前值的副本**。

- **示例**：

  ```javascript
  // 键路径
  vm.$watch('a.b.c', function (newVal, oldVal) {
    // 做点什么
  })
  
  // 函数
  vm.$watch(
    function () {
      // 表达式 `this.a + this.b` 每次得出一个不同的结果时
      // 处理函数都会被调用。
      // 这就像监听一个未被定义的计算属性
      return this.a + this.b
    },
    function (newVal, oldVal) {
      // 做点什么
    }
  )
  ```

  `vm.$watch` 返回一个取消观察函数，用来停止触发回调：

  ```javascript
  var unwatch = vm.$watch('a', cb)
  // 之后取消观察
  unwatch()
  ```

- **选项：deep**

  为了发现对象内部值的变化，可以在选项参数中指定 `deep: true` 。注意监听数组的变动不需要这么做。

  ```
  vm.$watch('someObject', callback, {
    deep: true
  })
  vm.someObject.nestedValue = 123
  // callback is fired
  ```

- **选项：immediate**

  在选项参数中指定 `immediate: true` 将**立即以表达式的当前值触发回调**：

  ```javascript
  vm.$watch('a', callback, {
    immediate: true
  })
  // 立即以 `a` 的当前值触发回调
  ```

在实践中我们通常以另一种方式写

```javascript
watch: {
    "attr1": {
        deep: true,
        immediate: false,
        handler(oldVal, newVal) {
            // TODO
        }
    },
    "attr2": {
        deep: true,
        immediate: true,
        handler(oldVal, newVal) {
            // TODO
        }
    }
}
```

总之监听变化，姐妹们选他！

#### vm.$set(target, propertyName/index, val)

- **参数**：

  - `{Object | Array} target`
  - `{string | number} propertyName/index`
  - `{any} value`

- **返回值**：设置的值。

- **用法**：

  这是全局 `Vue.set` 的**别名**。

- **参考**：[Vue.set](https://cn.vuejs.org/v2/api/#Vue-set)

这东西就是用来添加响应式属性的。当然，被添加属性的对象也得是响应式的，毕竟 vue 无法探测普通的新增属性。通过此属性我们可以动态的添加

### 实例方法/事件

#### vm.$on(event, callback)

这东西我就不再细讲了吧。之前写过一篇博客，感兴趣的胖友可以看一下原文地址：[《Vue中的$emit、$on和v-on》]([https://burning-shadow.github.io/2019/03/22/Vue%E4%B8%AD%E7%9A%84$emit%E3%80%81$on%E5%92%8Cv-on/](https://burning-shadow.github.io/2019/03/22/Vue中的$emit、$on和v-on/))

#### vm.$once(event, callback)

这东西和`$on`差不多，只不过他只触发一次，完成后移除监听器

#### vm.$off([event, callback])

- **参数**：

  - `{string | Array<string>} event` (只在 2.2.2+ 支持数组)
  - `{Function} [callback]`

- **用法**：

  移除自定义事件监听器。

  - 如果没有提供参数，则移除所有的事件监听器；
  - 如果只提供了事件，则移除该事件所有的监听器；
  - 如果同时提供了事件与回调，则只移除这个回调的监听器。

#### vm.$emit(eventName, [...args])

这东西和`v-on`、`$on`组合着用，一个愿打一个愿挨蛮快乐。老规矩，往上边看。





OK，fine，中篇暂时告一段落了。江湖见弟兄们。