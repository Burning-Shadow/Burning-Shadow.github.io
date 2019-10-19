---
title: reactJS生命周期初探
date: 2019-09-20 13:52:07
categories: React
tags:
 - reactJS
 - 生命周期
---

​		类似于 VueJS，ReactJS 亦有自己的生命周期。

​		不同于 Vue 的 8 个阶段（`beforeCreate`、`created`、`beforeMount`、`mounted`、`beforeUpdate`、`update`、`beforeDestory`、`destoryed`），React 生命周期只分为三部分：初始化阶段、行中阶段、销毁阶段，而在其后再进行细分。

​		由于 Vue 在学习的过程中并未将此重点讲述，所以我们此次将着重从其定义、作用、与组件间的联系、及各个钩子函数的详解等方面对 ReactJS 生命周期进行介绍

<!--more-->

### 什么是生命周期

reactJS 特点：

> 去除所有的手动 DOM 操作（JSX）
>
> 组件将状态和结果一一对应起来

 		而其中最重要的部分，组件，其本质是 <i style="color: red;">状态机</i> ，输入确定，输出一定确定。即，一个 `state` 对应一个 `render` 的结果。

​		而状态发生 <i style="color: red">转换</i> 时则会触发不同的钩子函数，从而让开发者有机会做出响应

### 生命周期

#### 初始化阶段

​		用组件（`Class`）代码初始化生成实例

##### `getDefaultProps`

> 获取实例的默认属性。注意，此钩子函数**只会在组件的第一个实例被初始化时调用，且只调用一次。**。
>
> 同一个组件的所有实例，其拿到的默认属性都是一样的（**实例之间<i style="color: red;">共享引用</i>**）
>
> 此钩子函数是在 `createClass` 阶段触发，即使实例构建失败仍会被调用

​		在使用过程中务必注意其返回的是引用还是值。因为引用类型数据指向的同一块内存区域，所以进行更改时更改的是内存区域中的数据，牵一发而动全身。所以一般来说我们都要对其进行一个样本的复制（比如数组的 `slice` 方法）

##### `getInitialState`

> 获取 <i style="color: red;">每个</i> 实例的初始化状态

##### `componentWillMount`

> 组件被装载之前（即将被渲染到页面上）
>
> `render` 之前最后一次 <i style="color: red;">修改状态</i> 的机会

##### `render`

> 组件在 render 中渲染为虚拟 DOM（JSX），然后再由 reactJS 将虚拟的 DOM 转换为真正的 DOM 并放置在页面中
>
> 只能访问 `this.props` 和 `this.state`
>
> 只有一个顶层组件，<i style="color: red;">不允许</i> 修改状态和 DOM 输出（若强行修改则无法在服务端使用，不利于 SSR）

##### `componentDidMount`

> 组件被装载（渲染）成真实 DOM 之后触发，可以修改 DOM
>
> 操作 DOM 只能在此处完成

#### 运行中阶段

​		运行中实例的状态可能发生改变，这些改变 <i style="color: red;">有可能</i> 导致组件重新渲染。当然也可能仅仅是发生数据上的改变而不重新渲染

##### `componentWillReceiveProps`

> 组件将要接收到属性前调用
>
> 父组件修改属性触发，<i style="color: red;">可以修改</i> 新属性、状态

##### `shouldComponentUpdate`

> 选择（判断）组件是否要更新
>
> 由于有些情况状态或属性的变化并不会导致组件的更新，所以通过此钩子函数我们可以人为手动的控制
>
> 即使 render 返回结果相同，reactJS 也会通过 render 和 diff 两部来判断组件是否需要更新，若配置了该项则无需 reactJS 进行判断，大大的节约了性能
>
> <i style="color: red;">返回 false 会阻止 render 调用</i> 

##### `componentWillUpdate`

>运行中组件更新时触发
>
><i style="color: red;">同 componentWillMount，一样无法修改属性状态</i> 

##### `render`

>组件在 render 中渲染为虚拟 DOM（JSX），然后再由 reactJS 将虚拟的 DOM 转换为真正的 DOM 并放置在页面中，其功效同初始化阶段的 render一样

##### `componentDidUpdate`

>render 结束之后，真正的 DOM 被渲染完后调用
>
><i style="color: red;">可以修改 DOM</i> 

#### 销毁阶段

​		父组件渲染时子组件会经历初始化阶段和运行中阶段，而假如父组件在某一状态下删除了子组件（`render` 结果中不包含子组件），则之前渲染好的子组件就已不再需要，reactJS 会将其销毁。

##### `componentWillUnmount`

> 销毁阶段真正完成之前调用，给开发者最后的机会执行最后的清理操作

#### 图例

​		然后我们用一张图来表现一下起对应的流程，也许可以帮助我们更好的理解

![react-alive-circle](https://ae01.alicdn.com/kf/Hc7300710040049adb770db5a7a9a43e4k.png)

### 属性的含义和用法

#### 属性的含义

> 属性：一个事物的性质与关系

