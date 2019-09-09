---
title: 重学Vue-Router（基础篇）
categories: 重学框架系列
tags: 
 - vue-router
---

之前只是单纯的学习，并未对`vue-router`做一个系统的总结。所以今天我试着参照官方文档对`vue-router`的一些常用 API 做一些总结作为日后复习所用。

<!--more-->

### 起步

`vue-router`的作用就是**将组件映射至路由上**，以告诉`vue-router`在哪里渲染他们。相较于原生 vueJS 组件化，`vue-router`使得变化可以显示到 url 上，更符合大家的编程习惯。

```html
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
```

```javascript
// 0. 如果使用模块化机制编程，导入Vue和VueRouter，要调用 Vue.use(VueRouter)

// 1. 定义 (路由) 组件。
// 可以从其他文件 import 进来
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
// 或者，只是一个组件配置对象。
// 我们晚点再讨论嵌套路由。
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

// 3. 创建 router 实例，然后传 `routes` 配置
// 你还可以传别的配置参数, 不过先这么简单着吧。
const router = new VueRouter({
  routes // (缩写) 相当于 routes: routes
})

// 4. 创建和挂载根实例。
// 记得要通过 router 配置参数注入路由，
// 从而让整个应用都有路由功能
const app = new Vue({
  router
}).$mount('#app')

// 现在，应用已经启动了！
```

### $router？$route

我们可以通过`this.$router`访问当前路由。也可以通过`this.$route`访问当前路由对象。那么二者有什么区别呢？

- `$router`是指**整个路由实例**。你可以操纵整个路由，通过`this.$router.push`向其中添加任意的路由对象
- `$route`是指**当前路由实例（`$router`）跳转到的路由对象**

通过此 API 我们可以实现一些针对于页面的操作

>`this.$router.go(num)`：向前/后跳转页面
>
>`this.$router.push('/')`：跳转到根路径



### 动态传参

动态传参一般用于传递动态参数。举个栗子，我们在博客页面在点击详情页时会动态传递一个参数（比如第1篇博客则传入编号为1）。

动态传参有两种形式：`params`和`query`

#### query

`query`传参有些类似于为后台以字符串形式传参，总之就是一个**丑**。

我们只需要在`.vue`文件中定义

```html
// query通过path切换路由
<router-link :to="{path: 'Detail', query: { id: 1 }}">前往Detail页面</router-link>
```

#### params

`params`传参则是需要在`route`中以冒号开头定义

```javascript
const User = {
  template: '<div>User</div>'
}

const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User }
  ]
})
```

而在`.vue`文件中则是在`:to`中定义

```html
<router-link :to="{name: 'Detail', params: { id: 1}">详情</router-link>
```

#### 区别

>**`path`和`params`无法同时生效**，否则`params`会被忽略掉。
>
>所以使用对象写法进行`params`传参时，要么就是`path`加冒号，要么就是像命名路由，通过`name`和`params`进行传参。
>
>而`query`却不受影响，有没有`path`都可以传参

这俩的区别也显而易见：

##### 丑

一个丑一个好看，这个我不多BB了吧

##### 切换路由的方式

````html
// query通过path切换路由
<router-link :to="{path: 'Detail', query: { id: 1 }}">前往Detail页面</router-link>
// params通过name切换路由
<router-link :to="{name: 'Detail', params: { id: 1 }}">前往Detail页面</router-link>
````

##### 接收参数的方式

**看清楚咯老弟！！！这是`this.$route`！（当前路由实例）不是`this.$router`（整个路由实例）**

```javascript
// query通过this.$route.query接收参数
created () {
    const id = this.$route.query.id;
}

// params通过this.$route.params来接收参数
created () {
    const id = this.$route.params.id;
}
```

##### route/index.js文件

`params`必须在路由中的`index.js`文件中定义

```javascript
<router-link :to="{name: 'Detail', params: { id: 1}">详情</router-link>

{      
    path: '/detail/:id',      
    name: 'Detail',      
    component: Detail    
},
```

而`query`传参则不需要

#### 响应路由参数的变化

从`/user/foo`跳转到`/user/bar`中时`user`组件会**被复用**。这也意味着**`user`组件的生命周期钩子不会再被调用**

所以为了解决这种方式我们可以用`watch`监听`$router`对象的变化

````javascript
const User = {
  template: '...',
  watch: {
    // 看清楚咯老弟！！！这是this.$route！（当前路由实例）不是this.$router（整个路由实例）
    '$route' (to, from) {
      // 对路由变化作出响应...
    }
  }
}
````

或者使用 2.2 新增的 API：`beforeRouteUpdate`（导航守卫）

```javascript
const User = {
  template: '...',
  beforeRouteUpdate (to, from, next) {
    // react to route changes...
    // don't forget to call next()
  }
}
```

#### 路由捕获

```javascript
{
  // 会匹配所有路径
  path: '*'
}
{
  // 会匹配以 `/user-` 开头的任意路径
  path: '/user-*'
}
```

注意，当我们使用了通配符`*`后则`$route.params`会自动添加一个`pathMatch`参数，它就是`*`被匹配的部分

### 编程式导航

#### router.push(location, onComplete?, onAbort?)

这东西咱们在《`$router?$route`》部分就提了一嘴。那么我们就来细看一下`this.$router.push`这个API

```javas
router.push(location, onComplete?, onAbort?)
```

举几个栗子

```javascript
// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: '123' }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})
```

注意一哈，如果提供了`path`，那么`params`则会被忽略。

> 我们前面讲到了，**`query`通过`path`切换路由，而`params`则通过`name`切换路由**

```javascript
const userId = '123'
router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
// 这里的 params 不生效
router.push({ path: '/user', params: { userId }}) // -> /user
```

#### router.go(n)

在`history`记录中前进或后退多少步（类似于`window.history.go(n)`）

#### router.replace(location, onComplete?, onAbort?)

这个除了不会向`history`中添加新纪录之外和`router.push`一模一样（替换到当前的`history`记录）

### 命名视图&命名路由

这东西，说白了就是俩字儿，好认。

#### 命名路由

命名路由咱们用的比较多，毕竟`params`是靠`name`来切换路由的。

```javascript
routes: [
    {
        path: '/user/:userId',
        name: 'anyNameIsOk',
        component: User
    }
]
```

不过注意一点：我们命名路由中的`name`属性和`.vue`文件中的`name`属性是不一样的！！！

给你们举个例子，我自己做的项目，跑跑，你可以去**`/route/index.js`**中看一下，为了区分我们将每个路由的`name`属性前面都加了其相应的权限（`windowAdmin`、`admin`、`superAdmin`），而在对应的组件（`component`）中没有一个和他们的`name`是一样的。但依然解决了报错问题。

![](https://pic.superbed.cn/item/5cd251643a213b0417f8754f)

**项目地址今后我完成后会贴在这里。**

##### params切换路由？

众所周知`params`传参是根据`name`属性进行路由切换的，**而此`name`属性就是命名路由中的`name`**。

诸位好汉别写成`.vue`文件中的`name`了哦。

#### 命名视图

而命名视图则是给我们的`<router-view>`起一个名字（`name`属性）。当一个路由需要匹配多个组件时就需要带上`name`属性咯

```html
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

### 提一嘴就够

#### 重定向

只要匹配到相应的路径，那么我们就从此路径重定向至另一个路径。一般结合`*`使用，将乱七八糟的网页重定向到主页上。

```javascript
const router = new VueRouter({
  routes: [
    { path: '/foo', component: Foo },
  	{ path: '/bar', component: Bar },
    { path: '*', redirect: '/'}
  ]
})
```

当然重定向的目标也可以是一个**命名路由**，甚至是一个**方法**

```javascript
// 目标为命名路由
{ path: '/a', redirect: { name: 'foo' }}

// 目标为方法
{ path: '/a', redirect: to => {
    // 方法接收 目标路由 作为参数
    // return 重定向的 字符串路径/路径对象
}}
```

#### 别名

**/a 的别名是 /b，意味着，当用户访问 /b 时，URL 会保持为 /b，但是路由匹配则为 /a，就像用户访问 /a 一样。**

```javascript
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})
```

### 路由组件传参

上面我们说到了`params`和`query`。不过这玩意儿本质上都是把参数放在 URL 上，通过改变 URL 进行的。如此即会造成参数和组件的高度耦合

而如果使用`propos`则可以摆脱 URL 的束缚（不改变 URL），提高组件的复用性。

```javascript
//路由配置:
const Hello = {
  props: ['name'], //使用rute的props传参的时候,对应的组件一定要添加props进行接收,否则根本拿不到传参
  template: '<div>Hello {{ $route.params}}和{{this.name}}</div>'
  //如果this.name有值,那么name已经成功成为组件的属性,传参成功
}


const router = new VueRouter({
mode: 'history',
  routes: [
    { path: '/', component: Hello }, // 没有传参  所以组件什么都拿不到
    { path: '/hello/:name', component: Hello, props: true }, //布尔模式: props 被设置为 true，此时route.params (即此处的name)将会被设置为组件属性。
    { path: '/static', component: Hello, props: { name: 'world' }}, // 对象模式: 此时就和params没什么关系了.此时的name将直接传给Hello组件.注意:此时的props需为静态!
    { path: '/dynamic/:years', component: Hello, props: dynamicPropsFn }, // 函数模式: 1,这个函数可以默认接受一个参数即当前路由对象.2,这个函数返回的是一个对象.3,在这个函数里你可以将静态值与路由相关值进行处理.
    { path: '/attrs', component: Hello, props: { name: 'attrs' }}
  ]
})
```

而在 HTML 中则正常使用组件即可。所有的逻辑传参部分全部在我们的路由设置（`router/index.js`）中完成，HTML（`.vue`）中无需做任何处理，并且也不会改变 URL。

比如这串代码：

```javascript
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})
```

通过`props`解耦后效果如下

```javascript
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User, props: true },

    // 对于包含命名视图的路由，你必须分别为每个命名视图添加 `props` 选项：
    {
      path: '/user/:id',
      components: { default: User, sidebar: Sidebar },
      props: { default: true, sidebar: false }
    }
  ]
})
```



### HTML5 History模式

开发时我们默认使用`hash`模式，以此来避免页面的重新加载，使得开发效率更高。

而`history`模式则是，只要 URL 发生更改都会向服务器发送请求，以此来实时刷新数据。

```javascript
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```

### 重点小结

- `$router`代表**整个路由实例**，`$route`则代表**当前路由实例**

- `<router-link :to="{ }">`等同于`this.$router.push()`

  - `path`和`params`是不能同时存在的

  - `params`参数都不会显示在 URL 地址栏中，除了在路由中通过`routes`进行配置的。所以用户刷新页面后`params`参数就会丢失
  - `query`参数不依赖于`route`，故刷新页面后亦会存在于地址栏中

-  路由组建传参中的函数模式可以让传递参数更加灵活，适用于需要逻辑控制的项目。

