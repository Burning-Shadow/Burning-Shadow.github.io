---
title: 重学Vue-Router（提升篇）
date: 2019-05-13 12:13:01
categories: 重学框架系列
tags:
 - vue-router
---

上一篇我们讲解了 Vue Router 的基本使用方法，而这次继续是官方 API 的解读，尽可能的会加入一些实践中所用到的案例

<!--more-->

### 路由对象属性介绍

看好咯老弟咱这个是针对`$route`而不是`$router`的

- **`$route.path`** 类型: `string` 字符串，对应当前路由的路径，总是解析为绝对路径，如 `"/foo/bar"`。
- **`$route.params`** 类型: `Object` 一个 key/value 对象，包含了动态片段和全匹配片段，如果没有路由参数，就是一个空对象。
- **`$route.query`** 类型: `Object` 一个 key/value 对象，表示 URL 查询参数。例如，对于路径 `/foo?user=1`，则有 `$route.query.user == 1`，如果没有查询参数，则是个空对象。
- **`$route .name`** 当前路由的名称，如果有的话。**这里建议最好给每个路由对象命名,方便以后编程式导航.不过记住name必须唯一!**
- **`$route.hash`** 类型: `string` 当前路由的 hash 值 (带 `#`) ，如果没有 hash 值，则为空字符串。
- **`$route.fullPath`** 类型: `string` 完成解析后的 URL，包含查询参数和 hash 的完整路径。
- **`$route.matched`** 类型: `Array<RouteRecord>` 一个数组，包含当前路由的所有嵌套路径片段的**路由记录** 。路由记录就是 `routes` 配置数组中的对象副本 (还有在 `children` 数组)。
- **`$route.redirectedFrom`** 如果存在重定向，即为重定向来源的路由的名字。

### 加载优化

#### 路由懒加载

作为单页面应用首屏需要加载的资源自然是非常多的。而如果按需加载则可以大大减少首屏时间。所以我们可以改变组件的引入形式（改为`promise`），**通过 Vue 的异步组件和 WebPack 的代码分割功能轻松实现懒加载**

```javasctipt
routes:[
  path:'/',
  name:'HelloWorld',
  component:resolve=>require(['@/component/HelloWorld'],resolve)
]
```

#### 组件的按组分块

把组件按组分块可以将路由下的所有组件都打包在同个异步块（`chunk`）中，并且在 F12 的`NetWork`中看到动态加载的组件名字

当然，选哟`WebPack`版本高于2.4且需要在`webpack.base.conf.js`里的`output`里面的`filename`下面加上`chunkFileName`

```javascript
output: {
 path: config.build.assetsRoot,
 filename: '[name].js',
 // 需要配置的地方
 chunkFilename: '[name].js',
 publicPath: process.env.NODE_ENV === 'production'
   ? config.build.assetsPublicPath
   : config.dev.assetsPublicPath
}
```

此时在引入组件时的写法需要使用命名`chunk`，一个特殊的注释语法来提供`chunk name`

```javascript
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
const Baz = () => import(/* webpackChunkName: "group-foo" */ './Baz.vue')
```

### 导航守卫

先从官网上复制粘贴一下**完整的导航解析流程**叭

1. 导航被触发。
2. 在失活的组件里调用离开守卫。
3. 调用全局的 `beforeEach` 守卫。
4. 在重用的组件里调用 `beforeRouteUpdate` 守卫 (2.2+)。
5. 在路由配置里调用 `beforeEnter`。
6. 解析异步路由组件。
7. 在被激活的组件里调用 `beforeRouteEnter`。
8. 调用全局的 `beforeResolve` 守卫 (2.5+)。
9. 导航被确认。
10. 调用全局的 `afterEach` 钩子。
11. 触发 DOM 更新。
12. 用创建好的实例调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数。

![](https://pic.superbed.cn/item/5cd410f33a213b04171a934d)

#### 守卫的三个参数

路由守卫中的每个守卫接收三个参数

- **to: Route**: 即将要进入的目标 [路由对象](https://router.vuejs.org/zh/api/#路由对象)
- **from: Route**: 当前导航正要离开的路由
- **next: Function**: 一定要调用该方法来 **resolve** 这个钩子。执行效果依赖 `next` 方法的调用参数。
  - **next()**: 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 **confirmed** (确认的)。
  - **next(false)**: 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 `from` 路由对应的地址。
  - **next('/') 或者 next({ path: '/' })**: 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。你可以向 `next` 传递任意位置对象，且允许设置诸如 `replace: true`、`name: 'home'` 之类的选项以及任何用在 [`router-link` 的 `to` prop](https://router.vuejs.org/zh/api/#to) 或 [`router.push`](https://router.vuejs.org/zh/api/#router-push) 中的选项。
  - **next(error)**: (2.4.0+) 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 [`router.onError()`](https://router.vuejs.org/zh/api/#router-onerror) 注册过的回调。

**确保要调用 next 方法，否则钩子就不会被 resolved。**

#### 全局守卫

##### 全局前置守卫

```javascript
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  // ...
})
```

##### 全局解析守卫

在 2.5.0+ 你可以用 `router.beforeResolve` 注册一个全局守卫。这和 `router.beforeEach` 类似，区别是在导航被确认之前，**同时在所有组件内守卫和异步路由组件被解析之后**，解析守卫就被调用。

##### 全局后置钩子

你也可以注册全局后置钩子，然而和守卫不同的是，这些钩子不会接受 `next` 函数也不会改变导航本身：

```javascript
router.afterEach((to, from) => {
  // ...
})
```

#### next()的详细使用说明

```javascript
// 对函数及next()的详细使用说明
router.beforeEach((to, from, next) => { 
  // 首先to和from 其实是一个路由对象,所以路由对象的属性都是可以获取到的(具体可以查看官方路由对象的api文档).
  // 例如:我想获取获取to的完整路径就是to.path.获取to的子路由to.matched[0].
  next();//使用时,千万不能漏写next!!!
  // next()  表示直接进入下一个钩子.
  // next(false)  中断当前导航
  // next('/path路径')或者对象形式next({path:'/path路径'})  跳转到path路由地址
  // next({path:'/shotcat',name:'shotCat',replace:true,query:{logoin:true}...})  这种对象的写法,可以往里面添加很多.router-link 的 to prop 和 router.push 中的选项(具体可以查看api的官方文档)全都是可以添加进去的,再说明下,replace:true表示替换当前路由地址,常用于权限判断后的路由修改.
  // next(error)的用法,(需2.4.0+) 
}).catch(()=>{
  //跳转失败页面
  next({ path: '/error', replace: true, query: { back: false }})
})
//如果你想跳转报错后,再回调做点其他的可以使用 router.onError()
router.onError(callback => { 
  console.log('出错了!', callback);
});
```



#### 路由独享守卫

在路由配置上（`route`）直接定义`beforeEnter`守卫，用法和对`router`的一样

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})
```

#### 组件内的守卫

这类钩子是写在组件内部的

- `beforeRouteEnter`进入路由前,此时实例还没创建,无法获取到`this`
- `beforeRouteUpdate (2.2)`路由复用同一个组件时
- `beforeRouteLeave`离开当前路由,此时可以用来保存数据,或数据初始化,或关闭定时器等等

```javascript
//在组件内部进行配置,这里的函数用法也是和beforeEach一毛一样
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
  }
}
```

### 路由元信息

作为一个啥都能干的标签`<meta>`可谓是让大家头疼不已。但由于其广泛的作用和强悍的能力使得我们仍在使用。而路由元信息则是可以在定义路由的时候配置`meta`字段

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      children: [
        {
          path: 'bar',
          component: Bar,
          // a meta field
          meta: { requiresAuth: true }
        }
      ]
    }
  ]
})
```

可是，既然我们可以给定义的路由配置`meta`，又怎么解决嵌套路由的问题？到时候`meta`用谁得好？



一**个路由匹配到的所有路由记录会暴露为 `$route` 对象 (还有在导航守卫中的路由对象) 的 `$route.matched` 数组。因此，我们需要遍历 `$route.matched` 来检查路由记录中的 `meta` 字段。**

```javascript
router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requiresAuth)) {
    // this route requires auth, check if logged in
    // if not, redirect to login page. 
    // matched是包含当前路由的所有嵌套路径片段的路由记录，而some则是一真具真
      
    // 若未登录则跳转到登录页
    if (!auth.loggedIn()) {
      next({
        path: '/login',
        // 通过query保存要跳转的路由路径，完成登陆后就可以直接跳转到登陆前要去的路由
        query: { redirect: to.fullPath }
      })
    } else {
      next()
    }
  } else {
    next() // 确保一定要调用 next()
  }
})
```

### 前进后退不变滚动条？

又要马儿跑又要马儿不吃草？`vue-router`可以做到，当然，仅在支持`history.pushState`的浏览器中生效。

```java
const router = new VueRouter({
  routes: [...],
  scrollBehavior (to, from, savedPosition) {
    // to: 目标路由对象
    // from：离开的路由对象
    // savedPosition：点击前进/后退的时候记录纸{x: ?, y: ?}，且只有通过浏览器的进退才会触发
    // return 期望滚动到哪个的位置 {x: number, y: number} / {selector: string, offset}: { x: number, y: number }}
    // 此处selector接受字符串的形式的hash
      
    if(savePosition) { //如果是浏览器的前进后退就,返回之前保存的位置
      return savePosition;
    }else if(to.hash) {//如果存在hash,就滚动到hash所在位置
      return {selector: to.hash}
    }else{
	  return {x:0,y:0}//否则就滚动到顶部
	}
  }
})
```

