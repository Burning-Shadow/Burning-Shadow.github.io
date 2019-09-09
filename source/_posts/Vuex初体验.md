---
title: Vuex初体验
categories: Vuex
tags:
 - vuex
---

### 全局引入
`main.js`中引入
```javascript
import Vuex from 'vuex
Vue.use(Vuex)
new Vue({
	el: '#app',
	router,
	components:{...},
	template: '...',
	store
})
```
<!--more-->

### 创建

- state 用来数据共享数据存储
- mutation 用来注册改变数据状态
- getters 用来对共享数据进行过滤操作
- action 解决异步改变共享数据
```javascript
const store = new Vuex.Store({
	// 全局状态
	state: {
		count: 0
	},
	
	// 修改state的必要途径
	mutations: {
		increment(state, n) {
			// 传入参数state是对state部分的引用
			state.count += n
		},
		decrement(state, n) {
			state.count -= n
		}
	},
	
	// 只读属性，存放过滤方法
	getters: {
		myCount(state){
			return `current count is ${state.count}`
		}
	},
	
	// 存放业务逻辑
	actions: {
		myIncrease(context, obj){
			context.commit('increment', 4)
		},
		myDecrease(context){
			context.commit('decrement', 2)
		}
	}
})
```
不过大家注意一点，`actions`中我们引用`mutations`中的方法时用到了 **`commit`，它就像Vue中的`$emit`一样，用于发布任务，而`mutations`中的方法则监听调用自己的地方，时刻准备为其服务**
### 组件引入

```javascript
imoprt { mapState } from 'vuex
```
- 我们通过`computed`属性进行实时监测（本例中使用解构赋值）。如此就可以在页面中使用`state`中的值了。
- 而方法的传递则需要我们进行`commit`调用。当然使用`this.methodName()`调用亦可
```html
<template>
	<button @click="increase">Add</button>
	<button @click="decrease">Minus</button>
</template>
<script>
import {mapMutations, mapActions, mapState, mapGetter}
export default{
	......
	computed: {
		...mapState(['count'])
	},
	methods: {
		// 声明引入actions和mutations中的方法
		...mapMutations(['increment', 'decrement']),
		...mapActions(['myIncrease', 'myDecrease']),
	
		async increse(){
			const products = await 
				this.$store.commit('increment', 1)
				// this.increment(1)		亦可
				// this.$store.state.count++
		
				// this.myIncrease({id: 123})
		},
		async decrese(){
			const products = await 
				this.$store.commit('decrement', 2)
				// this.decrement(2)		亦可
				// this.$store.state.count--
	
				// this.myDecrease()
		}
	}
}
</script>
```
上面列举的三种方法传递的方法中，我们发现通过`this.$store.state.count++`方式调用时无法令`Detected Vue`记录，因为他跳过了`mutation`直接对`state`数据进行操作，自然无法进行记录

- 如果你仅仅是想完成count的加减，我推荐你使用`action`。毕竟是逻辑代码，更方便读者理解。真正的使用方式比如通过此方式从后台调用数据
```javascript
actions: {
	async myIncrease(context) {
		context.commit('increment')	// 调用mutations中的increment方法
		const products = await axios.get(......)
		return products
	}
}

// component
methods: {
	...mapActions(['myIncrease', 'myDecrease']),
	async increase(){
		const products = await this.myIncrease();
		// do Some Thing
	}
}
```
### 封装
我们将Vuex的实例放在`main.js`中显然不是明智之举，所以我们不妨新建一个`store`文件夹，将其主题存至`store`中的`index.js`中（如此一来我们只需要引入`./store/index`即可完成）
```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
	// 全局状态
	state: {
		count: 0
	},
	
	// 修改state的必要途径
	mutations: {
		increment(state, n) {
			// 传入参数state是对state部分的引用
			state.count += n
		},
		decrement(state, n) {
			state.count -= n
		}
	},
	
	// 只读属性，存放过滤方法
	getters: {
		myCount(state){
			return `current count is ${state.count}`
		}
	},
	
	// 存放业务逻辑
	actions: {
		myIncrease(context, obj){
			context.commit('increment', 4)
		},
		myDecrease(context){
			context.commit('decrement', 2)
		}
	}
})

export default store
```
```javascript
// main.js
import store from './store/index'

new Vue({
	el: '#app',
	router,
	components: {....},
	template: '....',
	store
})

```
之后记得在`main.js`之中引用哦
```javascript
const module1 = {
	state: {....},
	mutations: {....},
	....
}
export default module1
```
同时，如果你的业务分为多种模块，那么也可以在`store`文件夹下建立相应的js文件，将其暴露出来即可
```javascript
import Module1 from './module1'
import Module2 from './module2'

Vue.use(Vuex)

const store = new Vuex.Store({
	modules:{
		Module1, Module2
	}
})

export default store
```
而同时，在组建之中我们需要用隐射调用
```javascript
computed:{
	...mapState({
		count: state=>{
			return state.app.count
		}
	})
}
```

![在这里插入图片描述](https://ww1.sinaimg.cn/large/007i4MEmgy1g1avrg3n5gj30kq0kqq3j.jpg)