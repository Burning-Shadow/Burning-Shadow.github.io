---
title: Vue中的$emit、$on和v-on
date: 2019-05-06 12:13:00
categories: Vue
tags: 
 - v-on
 - $emit
 - $on
---

由于本人才疏学浅本文章参考了诸多大神的想法，会在文末贴上原文地址

### $on && $ emit
>`$emit(eventName, [...args])`：触发事件
>`$on(eventName, callBack)`：监听事件

监听当前实例上的自定义事件，可以由`vm.$emit`触发。回调函数会接受所有传入事件出发函数的额外参数。



如果把`Vue`看成一个家庭（相当于一个单独的`components`)，女主人一直在家里指派`($emit)`男人做事，而男人则一直监听`($on)`着女士的指派`($emit)里eventName`所触发的事件消息，一旦 `$emit` 事件一触发，`$on` 则监听到 `$emit` 所派发的事件，派发出的命令和执行派执命令所要做的事都是一一对应的。

<!--more-->

```javascript
<template>
  <div>
      <p @click='emit'>{{msg}}</p>
      <p @click='emitOther'>{{msg2}}</p>
  </div>
</template>

<script>
export default {
  name: 'demo',
  data () {
      return {
         msg : '点击后女人派发洗东西事件',
         msg2 : '点击后女人派发开车事件',
      }
  },
  created () {

      this.$on(['wash_Goods','drive_Car'],(arg)=> {
          console.log('事真多')
      })
      this.$on('wash_Goods',(arg)=> {
          console.log(arg)
      })
      this.$on('drive_Car',(...arg)=> {
          console.log(BMW,Ferrari)
      })
  },
  methods : {
      emit () {
         this.$emit('wash_Goods','fish')
      },
      emitOther () {
         this.$emit('drive_Car',['BMW','Ferrari'])
      }
  }
}
</script>
```
如本例所示，我们可以将要监听的事件统一写到一个数组中，方便我们做一个类似于过滤器的统一处理。比如例子里的`this.$on(['wash_Goods','drive_Car'],(arg)=> { console.log('事真多')})`就类似于过滤器之类的机制，可以对“麻烦的女人”指派（`emit`）的事件进行一个统一的监听（`on`）并处理（`****真麻烦！`）。



上述例子中就是相当于通过`this.$emit('wash_Goods', 'fish')`给男人一个手册，告诉男人东西放在哪里，需要什么工具等等。



我们平时在开源库里使用的框架中都有无限下拉组件，那么我们一起跟随大神“混元霹雳手”的视角去看一下这类组件的实现
```javascript
<template>
    <div>
        <slot name="list"></slot>

        <div class="list-donetip" v-show="!isLoading && isDone">
            <slot>没有更多数据了</slot>
        </div>

        <div class="list-loading" v-show="isLoading">
            <slot>加载中</slot>
        </div>
    </div>
</template>

<script type="text/babel">

    export default {
        data() {
            return {
                isLoading: false,
                isDone: false,
            }
        },
        props: {
            onInfinite: {
                type: Function,
                required: true
            },
            distance : {
                type : Number,
                default：100
            }
        },
        methods: {
            init() {
                this.$on('loadedDone', () => {
                    this.isLoading = false;
                    this.isDone = true;
                });

                this.$on('finishLoad', () => {
                    this.isLoading = false;
                });
            },
            scrollHandler() {
                if (this.isLoading || this.isDone) return;
                let baseHeight = this.scrollview == window ? document.body.offsetHeight : this.scrollview.offsetHeight
                let moreHeight = this.scrollview == window ? document.body.scrollHeight : this.scrollview.scrollHeight;
                let scrollTop = this.scrollview == window ? document.body.scrollTop : this.scrollview.scrollTop

                if (baseHeight + scrollTop + this.distance > moreHeight) {
                    this.isLoading = true;
                    this.onInfinite()
                }
            }
        },
        mounted() {
            this.scrollview = window
            this.scrollview.addEventListener('scroll', this.scrollHandler, false);
            this.$nextTick(this.init);
        },
    }
</script>
```
对下拉组件加载加更的组件进行了一个简单的封装：

```
data 参数解释：
```

- isLoading `false 代表正在执行下拉加载获取更多数据的标识`，`true代表数据加载完毕`
- isDone `false 代表数据没有全完加载完毕`，`true 代表数据已经全部加载完毕`

```
props 参数解释：
```

- onInfinite `父组件向子组件传入当滚动到底部时执行加载数据的函数`
- distance `距离滚动到底部的设定值`

#### 从此组件中，我们进行每一步的分析

- 在`mounted `的时候，对`window`对像进行了一个滚动监听，监听的函数为`  scrollHandler`
  - 当`isLoading，isDone`任何一个为true时则退出 

    - `isloading`为`true`时防止多次同样加载，必须等待加载完毕
    - `isDone`为`true`时说明所有数据已经加载完成，没有必要再执行`scrollHandler`

  - 同时在$nextTick中进行了初始化监听 

    - `loadedDone` 一旦组件实例$emit('loadedDone')事件时，执行回调，放开加载权限
    - `finishLoad` 一旦组件实例$emit('finishLoad')事件时，执行回调，放开加载权限

- 再看看 scrollHandler函数里发生了什么 

  - `if (this.isLoading || this.isDone) return;` 一旦一者为true，则退出，原因在mounted已经叙述过了

  - ```
    if (baseHeight + scrollTop + this.distance > moreHeight)
    ```

     当在window对象上监听scroll事件时，当滚动到底部的时候执行 

    - `this.isLoading = true;`防止重复监听
    - `this.onInfinite()`执行加载数据函数

##### 父组件中调用`infinite-scroll`组件

```javascript
<template>
      <div>
          <infinite-scroll :on-infinite='loadData' ref='infinite'>
               <ul slot='list'>
                  <li v-for='n in Number'></li>
               </ul>
          </infinite-scroll>
      </div>
</template>

<script type="text/babel">
import 'InfiniteScroll' from '.......' //引入infinitescroll.vue文件
    export default {
         data () {
           return {
              Number : 10
           }
         },
         methods : {
           loadData () {
             setTimeout(()=>{
                this.Number = 20
                this.$refs.infinite.$emit('loadDone')
             },1000) 
           }
        }
    }
</script>
```

当滑到底部的时候，infinite-scroll 组件组件内部会执行传入的`:on-infinite='loadData'`函数 同时在内部也会把 Loading 设置为 true，防止重复执行。

在这里用`this.$refs.infinite`拿到`infinite-scroll`组件的实例，同时触发事件之前在组件中 `$on` 已经监听着的事件，在一秒后进行改变数据，同时发出`loadDone`事情，告诉组件内部去执行`loadDone`的监听回调，数据已经全部加载完毕，设置`this.isDone = true；` 一旦`isDone`或者`isLoading`一者为`true`，则一直保持`return退出状态`。

**$emit 和 $on 必须都在实例上进行触发和监听。**

### 组件解释

至于解释部分嘛，有兴趣的boy可以移步另一篇博客[《Vue中实现下拉加载组件》]([https://burning-shadow.github.io/2019/03/21/Vue%E4%B8%AD%E5%AE%9E%E7%8E%B0%E4%B8%8B%E6%8B%89%E5%8A%A0%E8%BD%BD%E7%BB%84%E4%BB%B6/](https://burning-shadow.github.io/2019/03/21/Vue中实现下拉加载组件/))

### v-on
> `v-on`作用在引入子组件后的父组件模板，用来监听子组件释放的事件
>
> `$on`和`$emit`则只能作用在事件名一一对应的**同一个组件实例中**


父组件中的`“v-on:”`之后绑定事件名+回调函数
```html
<demo v-on:eventName="callBack"></demo>
```
而子组件中则通过绑定触发事件的函数来将消息传回至父组件中（以此来触发父组件的回调函数）

>个人理解是，所谓的信息传递只不过是事件被触发后进行的某事件名的传递，以告诉父组件此事件被触发，之后父组件则得到此消息（主要是事件名）确定了此事件绑定的回调函数，并对其进行调用
```html
<template>
	<button @click="emit">点我给父组件传递消息</button>
</template>
export default {
	name: 'demo',
	methods: {
		emit() {
			this.$emit('eventName')
		}
	}
}
```

我们举个栗子

```html
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
<script>
Vue.component('button-counter', {
  template: '<button v-on:click="incrementCounter">{{ counter }}</button>',
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
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
</script>
```

如此例所示，在`counter-event-example`父组件里，声明了两个`button-count`的实列，**通过 `data` 用闭包的形式，让两者的数据都是单独享用的**，而且`v-on` 所监听的 `eventName` 都是当前自己实列中的 `$emit` 触发的事件，但是回调都是公用的一个 `incrementTotal` 函数，因为个实例所触发后都是执行一种操作！
PS:
- `v-on`声明在父组件中，你可以理解为一直抱着手机等你女神回消息的你：`<waiting v-on:call-repair="repair"></waiting>`。
- `emit`则在子组件中，一般通过某些动作触发调用对应函数，然后完成对信息的传递。比如你女神需要你给她修电脑的时候，她会在你俩尘封已久的聊天框上`click`一下：`<div @click="callRepair">会修电脑的人</div>`，而她的`methods`中必定有一个名为`call-repair`的方法，通过`this.$emit()`【没错，人家什么东西（参数）都不会给你，只会单纯的派遣任务，让你知道`call-repair`事件被触发，需要你调用你的`repair`方法】让你触发`repair`方法完成操作。




说到最后话题有点悲伤，不过也告诉我们相处应该讲究方法，如果你一开始将其视为自己的“女神（子组件）”百般呵护，单向输出，也怪不得人家调用`$emit`，只在用得到你的时候触发。

下一章我们会讲到更加“健康”的关系：`v-model`。双向沟通，相濡以沫才是正确的方式

最后，感谢掘金的“混元霹雳手”大大，他的博客给了我非常多的灵感，并且我引用了他的2个例子。[大佬博客传送门在这里](https://juejin.im/user/580327ee0e3dd900570cf3ab)。

今天就到这里啦
![在这里插入图片描述](https://ww1.sinaimg.cn/large/007i4MEmgy1g1avrg3n5gj30kq0kqq3j.jpg)