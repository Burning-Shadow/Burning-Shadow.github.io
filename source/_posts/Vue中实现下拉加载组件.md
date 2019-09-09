---
title: Vue中实现下拉加载组件
categories: Vue Template
tags: 
- Vue Template
---

#### 要点

作为一个类瀑布流型的组件，相较其他组件而言我们需要：

- 监听滚动事件
- 触底加载
  - 触底时触发事件向父组件请求信息
  - 若父组件返回信息则动态加载，并增加组件高度
  - 若父组件中未返回信息则显示“到底啦”
  - 若正在加载（网速较慢）则显示“加载中...”

<!--more-->

#### 代码

下面我们来一起看看这个组件8

```javascript
// infinite-scroll组件
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

#### 参数解释

- `data`：
  - `isLoading`代表是否处于加载阶段，是则显示“加载中”，如果可能再加个动画就更好啦
  - `isDone`则负责处理逻辑。加载完毕后将其设置为`true`。（防止多次加载）
- `props`：
  - `onInfinite`由父组件传递给我们，负责加载数据
  - `distance`也是由父组件传递给我们，用来由父组件设定滚动到底部的值（滚动到）
- `mounted`：对`window`执行一个滚动监听（`isLoading`和`isDone`任何一个为`true`时退出）
  - `scrollHandler`监听滚动，查看是否到达触发`onInfinite`的条件（到底了）
  - `loadedDone`：执行`$emit('loadedDone')`时执行回调，开放加载权限
  - `finishLoad`：执行`$emit('finishLoad')`时执行回调，开放加载权限

我不知道别人怎么想。但是我最早是理解不了为什么子组件中要加一个`<slot>`

后来发现如此即可使得父组件可以实现自定义样式，不再局限于子组件所套用的单一模板。作为一个前端开发DIY什么的真的是太`cool`了。



#### 父组件调用

```javascript
// 父组件中调用infinite-scroll组件
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
                this.$refs.infinite.$emit('loadedDone')
             },1000) 
           }
        }
    }
</script>
```

触底时组件会执行传入的`loadData`函数。同时在内部也会把`loading`设为`true`，防止重复执行。

同时通过`this.$refs.infinite`拿到`infinite-scroll`组件的实例，同时出发其之前在组件（父组件）中`$on`

监听的`loadedDone`使劲按，1s后改变数据，同时派发`loadedDone`，高速组建内部执行`loadedDone`的监听回调，完成数据加载。

