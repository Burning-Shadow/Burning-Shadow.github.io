---
title: Vue杂记
categories: Vue
tags: 
 - 路由传参
 - $nextTick
 - filter
 - watch
 - 性能提升
---

此章主要记录一些 Vue 项目所涉及到的一些简单问题

<!--more-->

### query && params

当向详情页面跳转时候难免会通过路由像后台传递一个参数。传参分为`query`传参和`params`+动态路由传参两种，所以我们一起来看看二者区别：

- `query`通过`path`切换路由。而`params`则通过`name`切换路由

  - ```
    // query通过path切换路由
    <router-link :to="{path: 'Detail', query: { id: 1 }}">前往Detail页面</router-link>
    // params通过name切换路由
    <router-link :to="{name: 'Detail', params: { id: 1 }}">前往Detail页面</router-link>
    ```

  - 

- `query`通过`this.$route.query`来接收参数，而`params`则通过`this.$route.params`来接收参数

  - ```
    // query通过this.$route.query接收参数
    created () {
        const id = this.$route.query.id;
    }
    
    // params通过this.$route.params来接收参数
    created () {
        const id = this.$route.params.id;
    }
    ```

  - 

- `query`传参的 URL 更像是通过 GET 方式向后台发送数据（`/detail?id=1&user=123&identity=1`），而`params`则更加美观一点（`/detail/123`）

- `params`动态路由传参必须要在路由中定义参数，然后在路由跳转的时候必须要加上参数，否则就是空白页面。

  - ```
    <router-link :to="{name: 'Detail', params: { id: 1}">详情</router-link>
    
    {      
        path: '/detail/:id',      
        name: 'Detail',      
        component: Detail    
    },
    ```

### 动态添加数据

由于框架限制，只有`data`中的数据才是响应式的，而我们动态添加进来的数据默认均为非响应式。

我们一般通过以下两种方式实现动态添加数据的响应式

- `Vue.set(object, key, value)`：适用于添加单个属性
- `Object.assign()`：适用于添加多个属性

```javascript
var vm = new Vue({
  data: {
    stu: {
      name: 'jack',
      age: 19
    }
  }
})

/* Vue.set */
Vue.set(vm.stu, 'gender', 'male')

/* Object.assign 将参数中的所有对象属性和值 合并到第一个参数 并返回合并后的对象*/
vm.stu = Object.assign({}, vm.stu, { gender: 'female', height: 180 })
```

以此方式我们可以强行把新加入的数据绑定在发布者-订阅者模式之中

### 异步 DOM 更新

Vue 异步执行 DOM 更新，监视所有数据改变，一次性更新 DOM。其优势在于避免重复操作 DOM。

但若我们需要用到更新后 DOM 中的数据，则需要通过 `Vue.nextTick(callback)`来进行调用

```javascript
methods: {
  fn() {
    this.msg = 'change'
    this.$nextTick(function () {
      console.log('$nextTick中打印：', this.$el.children[0].innerText);
    })
    console.log('直接打印：', this.$el.children[0].innerText);
  }
}
```

这个在实践中的例子就是编写手机端的一个软件时点击跳转路由却总是获取当前的路由，最后通过高人指点才通过`this.$nextTick`操作完成了加载

```javascript
this.$nextTick(()=>{
    // 在vue完成渲染任务之后的行为
    console.log(this.selected)
    name: this.selected
})
```

### 性能提升

#### v-pre

Vue 会跳过这个元素和其子元素的编译过程。可以用来显示原始`Mustache`标签。跳过大量没有指令的节点会加快编译

```html
<span v-pre>{{ this will not be compiled }}</span>
```

#### v-once

Vue 只渲染该元素和组件一次，随后的重新渲染，元素/组件**及其所有的子节点**将被视为静态内容并跳过

```html
<span v-once>This will never change: {{msg}}</span>
```

### 过滤器

这东西用于**文本数据格式化**，可以用在**双括号**和**`v-bind`表达式**上

#### 全局过滤器

全局过滤器我们一般用于过滤一些比较常用的数据，最经典的一个例子就是时间。我们一般将全局过滤器定义在`main.js`中

```html
<div>{{ dateStr | date }}</div>
<div>{{ dateStr | date('YYYY-MM-DD hh:mm:ss') }}</div>

<script>
  Vue.filter('date', function(value, format) {
    // value 要过滤的字符串内容，比如：dateStr
    // format 过滤器的参数，比如：'YYYY-MM-DD hh:mm:ss'
  })
</script>
```

项目中的一个例子就是利用插件将时间格式转码后显示

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

这样我们就可以在全局任意一个地方使用注册的全局过滤器了

#### 局部过滤器

只在所定义的实例之中起作用

```javascript
{
  data: {},
  // 通过 filters 属性创建局部过滤器
  // 注意：此处为 filters
  filters: {
    filterName: function(value, format) {}
  }
}
```

### 事件修饰符

我们可以通过一些事件修饰符来节省大量逻辑代码

- `.stop`
- `.prevent`
- `.capture`
- `.self`
- `.once`
- `.passive`

```html
<!-- 阻止单击事件继续传播 -->
<a v-on:click.stop="doThis"></a>

<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- 修饰符可以串联 -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>

<!-- 添加事件监听器时使用事件捕获模式 -->
<!-- 即元素自身触发的事件先在此处理，然后才交由内部元素进行处理 -->
<div v-on:click.capture="doThis">...</div>

<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
<!-- 即事件不是从内部元素触发的 -->
<div v-on:click.self="doThat">...</div>
```

### watch、computed & method

#### method

`method`就是普通意义的`function`区域。在其中定义我们可能会使用到的方法

#### computed

**具有缓存性，页面重新渲染值不变化，计算属性会立即返回之前的计算结果，不会重复计算**

```
el: '#app',
data: {
    firstname: 'jack',
    lastname: 'rose'
},
computed: {
    fullname() {
        return this.firstname + '.' + this.lastname
    }
}
```

> 赖收集、动态计算的流程：
>
> ```
> 1. data 属性初始化 getter setter
> 2. computed 计算属性初始化，提供的函数将用作属性 vm.reversedMessage 的 getter
> 3. 当首次获取 reversedMessage 计算属性的值时，Dep 开始依赖收集
> 4. 在执行 message getter 方法时，如果 Dep 处于依赖收集状态，则判定 message 为 reversedMessage 的依赖，并建立依赖关系
> 5. 当 message 发生变化时，根据依赖关系，触发 reverseMessage 的重新计算
> ```



#### watch

`watch`是 Vue 所提供的更通用的，用以观察相应实例数据变动的方式。其**键是要观察的表达式**，**值是对应的回调函数**。

可以理解为：观察一个动作。

**不具有缓存性。页面重新渲染时值不变化亦会重新计算**

通常其作用`computed`就可以完成，并且还省性能

```javascript
new Vue({
  data: { a: 1, b: { age: 10 } },
  watch: {
    a: function(val, oldVal) {
      // val 表示当前值
      // oldVal 表示旧值
      console.log('当前值为：' + val, '旧值为：' + oldVal)
    },

    // 监听对象属性的变化
    b: {
      handler: function (val, oldVal) { /* ... */ },
      // deep : true表示是否监听对象内部属性值的变化 
      deep: true
    },

    // 只监视user对象中age属性的变化
    'user.age': function (val, oldVal) {
    },
  }
})
```

### axios中的拦截器

拦截器会拦截发送的每一个请求。请求发送之前执行`request`的函数，请求发送之后执行`response`中的函数

```javascript
// 请求拦截器
axios.interceptors.request.use(function (config) {
    // 所有请求之前都要执行的操作
    return config;
}, function (error) {
    // 错误处理
    return Promise.reject(error);
});

// 响应拦截器
axios.interceptors.response.use(function (response) {
    // 所有请求完成后都要执行的操作
    return response;
  }, function (error) {
    // 错误处理
    return Promise.reject(error);
});
```

### 非父子组建通讯

#### 时间总线

简单的场景下可以用一个空的 Vue 实例作为事件总线

```javascript
var bus = new Vue()

// 在组件 B 绑定自定义事件
bus.$on('id-selected', function (id) {
  // ...
})
// 触发组件 A 中的事件
bus.$emit('id-selected', 1)
```

### 获取组件/DOM

尽管 Vue 不推荐用户直接操作 DOM，但还是留下了相关的 API—— `refs`

```
<div id="app">
  <div ref="dv"></div>
  <my res="my"></my>
</div>

<!-- js -->
<script>
  new Vue({
    el : "#app",
    mounted() {
      this.$refs.dv //获取到元素
      this.$refs.my //获取到组件
    },
    components : {
      my : {
        template: `<a>sss</a>`
      }
    }
  })
</script>
```



### 后记

好啦暂时总结这么多，以后项目中出现新的风暴再补充8。

