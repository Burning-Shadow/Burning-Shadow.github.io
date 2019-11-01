---
title: Vuex进阶
categories: 状态管理库
tags:
 - 状态管理库
 - vuex
---

在 Vuex 初体验中我们介绍了 Vuex 的基本使用方法。而一些特殊的写法以及使用方式我们还是得老老实实的刷一遍官方文档。

<!--more-->

### State

`state`就是状态数据，而 Vuex 管理的就是状态，其他的`Actions`、`Mutations`等都是辅助实现对状态的管理的。我们可以通过`this.$store.state`直接获取状态，

通常我们通过`mapState`函数将引入的`state`放至`computed`属性之中，使得其在有缓存的情况下保持动态更新。

#### mapState

当一个组件需要获取多个状态的时候，将其都声明为计算属性会有些重复和冗余，所以我们可以使用`mapState`辅助函数帮助我们生成计算属性

```javascript
// 在单独构建的版本中辅助函数为 Vuex.mapState
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',

    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```

若映射的计算属性的名称与`state`的子节点名称相同时，亦可给`mapState`传一个字符串数组

```javascript
computed: mapState([
    // 映射 this.count 为 store.state.count
	'count'
])

// 当然我们也可以写作如此形式
computed: {
    ...mapState(['count'])
}
```

### Getters

这东西基本是用于取数据，但是不一样的是我们可以使用它给取到的`state`进行处理，**可以理解为一个`filter`**。同样的，它有些类似 vue 中的`computed`属性，一旦依赖发生改变则会被重新计算，否则使用缓存。

我们可以通过`this.$store.getters.valueName`对派生出来的状态进行访问，或者直接使用`mapGetters`将其映射到`computed`中

#### mapGetters

```javascript
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}
```

若是希望在调用处重新起名则可以使用对象形式

```javascript
mapGetters({
  // 映射 `this.doneCount` 为 `store.getters.doneTodosCount`
  doneCount: 'doneTodosCount'
})
```

### Mutations

**`Mutations`译为“变化”**。顾名思义，务必通过提交（`commit`）`mutation`来出发状态的更新。双方约定一个事件名，并通过载荷（`payload`）完成参数的传递

我们在`mutations`中定义改变`state`中数据的方法

```javascript
mutations: {   //放置mutations方法
	increment(state, payload) {
		//在这里改变state中的数据
		state.count = payload.number;
	}
}
```

我们可以通过`commit`提交相应的`mutation`

```javascript
this.$store.commit('increment', {
    amount: 10
})

// 换一种写法也可以
this.$store.commit({
    type: 'increment',
    amount: 10
})
```

#### mapMutations

除此了使用`this.$store.comit('eventName')`的方式提交之外，还有另外一种方式：使用`mapMutations`辅助函数将数组中的`methods`映射为`this.$store.commit`：

```javascript
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

      // `mapMutations` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      // 这就是对象写法了
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
  }
}
```

如此，通过类似于事件处理函数（`handler`）和事件触发（`emit`）之间的关系，即可以完成`mutations`的操作

#### mutation-types

**考虑到触发的mutation的type必须与mutations里声明的mutation名称一致，比较好的方式是把这些mutation都集中到一个文件（如mutation-types）中以常量的形式定义，在其它地方再将这个文件引入，便于管理。而且这样做还有一个好处，就是整个应用中一共有哪些mutation type可以一目了然。**就像下面这样：

```javascript
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'

// store.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = new Vuex.Store({
  state: { ... },
  mutations: {
    // 我们可以使用 ES2015 风格的计算属性命名功能来使用一个常量作为函数名
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})
```

> Ps：`mutation`必须是同步函数，其目的是使得`dev-tools`可以捕捉到每一条`mutation`的前后状态的快照
>
> 原作者贴了相关帖子的链接，我也沾一下吧：[知乎传送门](https://www.zhihu.com/question/48759748)

### Actions

`Actions`按照我个人的理解就是用于处理逻辑代码。我们将操作数据的方法定义在`Mutations`中，将逻辑部分放置在`Actions`中。可以理解为：**`action`处理函数所作的事情就是`commit mutation`**。

更棒的是`Actions`可以包含异步操作。



其接受的参数如下

- `context`：与`store`实例具有相同方法和属性。我们可以调用`context.commit`提交一个`mutation`或者通过`context.state`和`context.getters`获取`state`和`getters`

#### 触发action

我们可以通过`dispatch`触发`action`方法。当然触发方式也有两种，大家挑个顺眼的用就行。

```javascript
// 载荷形式分发
this.$store.dispatch('incrementAsync', {
    amount: 10
})

// 以对象形式分发
this.$store.dispatch({
    type: 'incrementAsync',
    amount: 10
})
```

#### mapActions

```javascript
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`

      // `mapActions` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
  }
}
```

#### 异步操作

另外，`this.$store.dispatch`可以处理被触发的`action`的处理函数返回的`Promise`，且`this.$store.dispatch`仍旧返回`Promise`

```javascript
// 所以诸如此类的异步操作...
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}

// ...我们可以写作
$this.store.dispatch('actionA').then(() => {
  // ...
})
```

同样的我们可以使用 ES6 新型的 API

```javascript
// 假设 getData() 和 getOtherData() 返回的是 Promise

actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // 等待 actionA 完成
    commit('gotOtherData', await getOtherData())
  }
}
```

我们来举个例子,看一个更加实际的购物车示例，涉及到**调用异步 API** 和**分发多重 mutation**：

```javascript
actions: {
  checkout ({ commit, state }, products) {
    // 把当前购物车的物品备份起来
    const savedCartItems = [...state.cart.added]
    // 发出结账请求，然后乐观地清空购物车
    commit(types.CHECKOUT_REQUEST)
    // 购物 API 接受一个成功回调和一个失败回调
    shop.buyProducts(
      products,
      // 成功操作
      () => commit(types.CHECKOUT_SUCCESS),
      // 失败操作
      () => commit(types.CHECKOUT_FAILURE, savedCartItems)
    )
  }
}
```



