---
title: Vue组件间通讯
categories: Vue
tags:
 - 父子组件间通讯
 - 兄弟组件间通讯
 - 跨组件通讯
---



之前也写过关于组建通讯的文章，但主要是讲父子组件之间的通讯。此文章当作总结，以备日后复习

（事实上是因为我最近比较懒所以拿出来推到个人博客页面，吃个老本吧~）

<!--more-->

### 一.  父子组件间通讯

#### 1.1  单向传递   props

`Vue`中我们可以通过`props`完成父向子单向传递数据的任务。如此即可保证子组件无法随意更改父组件的数据，保证安全性

```html
<!-- 父组件 -->
<template>
  <div>
    <ChildCom :str='str' :func='func' :home='this'></ChildCom>
  </div>
</template>
export default {
  data() {
    return {
      str: 'I come from fatherComponent'
    };
  },
  components: {
    ChildComponent,
  },
  methods: {
    func () {
      alert('I am the methods from fatherComponent');
    },
  },
};
```

父组件通过`v-bind:`动态传递自己`data`和`methods`中的 数据及函数。**我们甚至可以将父组件的作用域指针`this`作为参数传递到子组件之中。**

而子组件则通过`props`数组进行统一接收

```html
<template>
 <div>
   <div class='str'>
     {{ str }}
   </div>
   <div class='buttons'>
     <button @click='func'>执行父组件的方法</button>
     <button @click='getParent()'>获取父组件的数据和方法</button>
   </div>
 </div>
</template>
<script>
export default {
 props: ['str', 'func'],
 methods: {
   getParent() {
     alert(this.home); // eslint-disable-line
     alert(this.home.str); // eslint-disable-line
     alert(this.home.func); // eslint-disable-line
   },
 } 
};
```

#### 1.2  子组件向父组件传递信息

于`props`无法更改父组件的数据，自然也无法向父组件传递数据。那么此时我们就引入新的通讯方式：监听`emit`

简单解释一下原理吧，父组件就想老父亲一样每天守着电话机（`v-on`）监听你所传回来的消息，而儿子（子组件）则通过`emit`管老爹要生活费（派发事件）

> v-on作用在引入子组件后的父组件模板，用来舰艇子组件派发的事件

```html
<!-- 父组件 -->
<template>
    <div id="counter-event-example">
      <p>{{ total }}</p>
      <button-counter @increment="incrementTotal"></button-counter>
      <button-counter @increment="incrementTotal"></button-counter>
    </div>
</template>
<script>
exoprt default {
  new Vue({
    el: '#counter-event-example',
    data: {
      total: 0
    },
    methods: {
      incrementTotal: function () {
        this.total += 1
      }
    },
    components: {
	  child
    }
  })
}
</script>
```

```html
<!--子组件 -->
<template>
    <button @click="incrementCounter">{{ counter }}</button>
</template>
<script>
export default{
  name: 'child',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    incrementCounter: function () {
      this.counter += 1
      this.$emit('increment')
    }
  }
}
</script>
```

- `v-on`声明在父组件中，时刻监听着子组件的`increment`事件。一旦此事件被触发则调用`incrementTotal()`函数
- `emit`则在子组件中使用，将`increment`事件派发，给父组件传递信号，令其调用`incrementTotal`完成自增。

> 当然，我们的emit还可以加一个参数，也就是父组件向子组件传递的参数

#### 1.3  直接更改props

看起来好像不太现实，不过实际操作我们算是投机取巧，绕开了原本的直接更改。我们可以将父组件中更改`data`中元素值的方法传递到子组件之中令其通过`props`接收。而后可以直接调用，通过子组件的调用完成更改父组件数据。我就不多赘述啦，各位好汉自己琢磨去吧。

### 二.  兄弟组件间通讯

#### 2.1  靠爹

之前咱们介绍了父子组件通讯，老爹靠`props`给儿子喊话，儿子则通过`emit`给老爹发消息传参。二者默契十足。但是好兄弟之间就没这么贴心了，所以需要老父亲夹在中间充当一个中转站。

```html
<!-- fatherComponent -->
<template>
  <div class='container'>
    <h2>父组件</h2>
    <button @click='stopCommunicate' v-if='showButton'>停止通讯</button>
    <div class='card-body'>
      <brother-card :messageSon='messageson' @brotherSaid='messageDaughter' class='card-brother'></brother-card>
      <sister-card :messageDaughter='messagedaughter' @sisterSaid='messageSon' class='card-sister'></sister-card>
    </div>
  </div>
</template>

<script>
import BrotherCard from './BrotherCard';
import SisterCard from './SisterCard';

export default {
  name: 'ParentCard',
  data() {
    return {
      messagedaughter: '',
      messageson: '',
    };
  },
  components: { BrotherCard, SisterCard },
  methods: {
    messageDaughter(message) {
      this.messagedaughter = message;
    },
    messageSon(message) {
      this.messageson = message;
    },
    showButton() {
      return this.messagedaughter && this.messageson;
    },
    stopCommunicate() {
      this.messagedaughter = '';
      this.messageson = '';
    },
  },
};
</script>
```

```html
<!-- brotherComponent -->
<template>
  <div>
    <div>
      <p>我是子组件：Brother</p>
      <button @click='messageSister'>给妹妹发消息</button>
      <div v-if='messageSon' v-html='messageSon'></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'BrotherCard',
  props: ['messageSon'],
  methods: {
    messageSister() {
      this.$emit('brotherSaid', 'Hi，妹妹');
    },
  },
};
</script>
```

```html
<!-- sisterComponent -->
<template>
  <div>
    <div>
      <p>我是子组件：Sister</p>
      <button @click='messageBrother'>给哥哥发消息</button>
      <div v-if='messageDaughter' v-html='messageDaughter'></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SisterCard',
  props: ['messageDaughter'],
  methods: {
    messageBrother() {
      this.$emit('sisterSaid', 'Hi，哥哥');
    },
  },
};
</script>
```

讲兄弟组件之间需要共享的数据提升至距其最近的父组件中以完成通讯，这就叫做**状态提升**

#### 2.2  EventBus

**通过使用事件中心，允许组件自由交流，无论组件处于组件树的哪一层。**

说白了和通过父组件完成兄弟组件间通讯的原理一样，只不过我们额外开辟了一个新的对象，使其无需再经过一层层的找寻完成通讯

当然，也有区别。我们再`main.js`中新建一个新的`vue`实例，将其命名为`eventBus`。使用`eventBus.$emit`代替`this.emit`，其余都一样。

```javascript
// main.js
export const eventHub = new Vue();
```

接着我们令`eventBus`实例成为子组件中派发事件的实例（使用`eventBus.$emit`代替`this.emit`）

```html
<template>
  <div>
    <p>我是Brother组件</p>
    <button @click='messageSister'>给妹妹发消息</button>

    <div v-if='fromSister' v-html='fromSister'></div>
  </div>
</template>

<script>
import { eventBus } from '../../main';

export default {
  name: 'BrotherComponent',
  data: () => ({
    fromSister: '',
  }),
  methods: {
    messageSister() {
      eventBus.$emit('brotherSaid', 'Hi，妹妹');
    },
  },
  /* eslint-disable */
  created() {
    eventBus.$on('sisterSaid', message => {
      this.fromSister = message;
    });
  },
};
</script>
```

在`created()`上添加`eventBus`，监听想要交互的兄弟组件的动作

```html
<template>
  <div>
    <p>我是Sister组件</p>
    <button @click='messageBrother' class='btn'>给哥哥发消息</button>
    <div v-if='fromBrother' v-html='fromBrother'></div>
  </div>
</template>

<script>
import { eventBus } from '../../main';

export default {
  name: 'SisterCard',
  data: () => ({
    fromBrother: '',
  }),
  methods: {
    messageBrother() {
      eventBus.$emit('sisterSaid', 'Hi，哥哥');
    },
  },
  /* eslint-disable */
  created() {
    eventBus.$on('brotherSaid', message => {
      this.fromBrother = message;
    });
  },
};
</script>
```

### 三.  随便怎么通讯

是的没错确实有这东西的存在，只不过是通过**共享数据的Vuex**或是**统一设置全局模式**实现。咱们上次刚说过Vuex所以趁热打铁先讲它好了

#### 3.1  Vuex

- `state`用来数据共享数据存储
- `mutation`用来注册改变数据状态
- `getters`用来对共享数据进行过滤操作
- `action`解决异步改变共享数据

##### 3.1.1  创建

```javascript
// store/index.js

import Vue from 'vue'
import Vuex from 'vuex'

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
		},
        async myIncrease(context) {
            context.commit('increment')	// 调用mutations中的increment方法
            const products = await axios.get(......)
            return products
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

**注意`commit`，它类似于`Vue`中的`$emit`，用于发布任务。而`mutations`中的方法则监听调用自己的地方，时刻准备为其服务。**

##### 3.1.2  引入

```javascript
imoprt { mapState, mapMutations, mapGetters, mapActions } from 'vuex
```

##### 3.1.3  使用

**在组件中使用前务必要引用（3.1.2方式，引入对应部分即可）**

###### state

```javascript
computed: {
    ...mapState(['dataName1, dataName2, ....'])
}
```

为了保证实时更新数据我们一般来讲将其`State`属性放在`computed`之中，随后即可在组件之中随意调用

###### mutations

更改数据时务必通过`mutations`中的方法，否则无法通过`chrome`插件`Detected Vue`观察变化情况

```javascript
methods: {
    ...mapMutations([method1, method2, ....])
}
```

`mutations`中的方法均为基本方法，用以进行对`state`中数据的更改。而`actions`中的方法则多为逻辑方法，他们对数据处理大都调用`mutations`，而自己专注于处理逻辑，比如封装`axios`函数之类的操作均可以写在里边

###### getters

`getters`中则是存储获取方法

###### actions

`actions`中存储逻辑方法

```javascript
methods: {
    ...mapActions(['myIncrease', 'myDecrease']),
    async increse(){
        const products = await 
        this.$store.commit('increment', 1)
    }
}
```

#### 3.2  全局模式

创建全局变量和方法，让其他组件之间共享数据存储

可以理解为暴露此对象的方法和属性使得组件可以调用。

```javascript
const store = {
  state: { numbers: [1, 2, 3] },
  addNumber(newNumber) {
    this.state.numbers.push(newNumber);
  },
};
```

而想要使用的组件引入其变量即可。说实在的不算什么高新技术，也就不再赘述了。

### 四. sync

#### 4.1前情

我印象中在 vue 初期好像有一个名为 `dispatch` 的 API，可以实现双向通信，而后作者感觉子组件更改父组件的值使得权力过于出格所以禁掉了相应的 API。随后随着 vue 语言的发展发现其亦有可取之处（功能上讲），且 `emit` 和 `v-on` 的组合也已面世，故通过 `emit` 和 `v-on` 所铸就的语法糖亦被加入了其中，而这个语法糖就名为 `sync`

#### 4.2 例子

这个东西平时想必大家也是闻多见少，那么我们就用 ElementUI 的例子为大家讲解。



element 中的`el-dialog`组件想必大家使用过，其中最明显的一行语句就是在定义时所述的：

```javascript
<el-button type="text" @click="dialogVisible = true">点击打开 Dialog</el-button>

<el-dialog
  title="提示"
  :visible.sync="dialogVisible"
  width="30%"
  :before-close="handleClose">
  <span>这是一段信息</span>
  <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
  </span>
</el-dialog>

<script>
  export default {
    data() {
      return {
        dialogVisible: false
      };
    },
    methods: {
      handleClose(done) {
        this.$confirm('确认关闭？')
          .then(_ => {
            done();
          })
          .catch(_ => {});
      }
    }
  };
</script>
```

`:visible.sync="dialogVisible"`。在 `dialog` 上我们定义了这句话，其绑定 `visible` 属性想必大家知道，那么 `.sync` 呢？

事实上在子组件中其会有相应的 `visible` 属性。通过 `$emit` 进行事件派发。

```html
<div v-if="dialogVisible">
    <span @click="close">Close</span>
</div>

<script>
    props: [visible],
    data() {
        return {
            dialogVisible: false
        }
    },
    watch: {
        visible(nVal) {
            this.dialogVisible = nVal;
        }
    },
    methods: {
        close() {
            this.dialogVisible = false;
            this.$emit("visible", false);
        }
    }
</script>
```



所以这玩意儿就相当于一个对于父组件的语法糖。样例也有啦，就是使得用户在操纵父组件时可以写出更语义化的代码，料儿还是一样的。





