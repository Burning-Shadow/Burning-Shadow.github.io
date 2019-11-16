---
title: Vue中有趣的自定义指令
date: 2019-08-15 21:58:00
categories: Vue
tags: 
 - directive
---

​	在使用过程中往往会面临一些稀奇古怪的需求。而此时实现是可以，但是也许会让代码的可维护性变得很差，同时复用性也大大降低。而在 Vue 官方文档中存在着这样一个 API：`Vue.directive(directiveName, func)`

<!--more-->

### 抛砖

​	讲人家的没什么说服力，那就举一个自己的例子。最近接了个埋点的活儿。

埋点嘛，很简单嘛，无非就是 `@click=func` ，然后再在 `func` 函数里写逻辑代码。。。。

​	但是这玩意儿看似得我维护……那咱就让他好改一点！

于是乎我们想到了 `document.addEventListener`。好就这么办！

```javascript
// ulog.js
export default {
  methods: {
    getULogCode(str){
      let code,
          i,
          reg = /^ULOG_/g;
      let arr = str.split(" ");
      
      if(arr.length === 0)	return null;
      for(i=0; i<arr.length; i++){
        if(reg.test(arr[i])){
          code = arr[i].slice(5);
          break;
        }
      }
      (i === arr.length)? (code = null): (code = code);
      return code;
    }
  },
  mounted(){
    this.$nextTick(()=>{
      document.addEventListener("click", e=>{
        let target = e.target || e.srcElement, 
          flag = 3,
          ULOG_code = this.getULogCode(target.className)|| null;
      
        while(ULOG_code === null && flag > 0){
          flag--;
          target = target.parentNode;
          ULOG_code = this.getULogCode(target.className) || null
        }
        
        if(!ULOG_code)	return;
        
        this.$axios().then().finally();
      })
    })
  }
}
```

讲配置项放在一个单独的配置文件中，再将其混入 App.vue

```javascript
// App.vue
import ulogMixin from "@/mixins/ulog";

export default {
  mixins: [ulogMixin]
}
```

诺，就这样，既然是通过监听点击事件且要好维护，那么我们就给相应的 DOM 节点以统一格式的类名，后边的要啥咱们根据产品老哥给的那张表来完善~



​	是不是觉得跑题了？这只是个引子！我们纵然可以通过配置类名来进行埋点。但是有没有想过，在部分组件里是禁止冒泡的！我们纵然可以取巧，将 `click` 改为 `mousedown`，但是我们却很难对其传递自定义参数。

### 引玉

自定义指令的使用途径一般在一些特殊情况。像是官网中就举了一个自动聚焦的例子。

[详情点击这里](https://cn.vuejs.org/v2/guide/custom-directive.html)

此时我们就应该去进行一些自定义指令的配置

```javas
// monitor.js

export default {
  inserted: function(el, binding) {
    document.addEventListener("mousedown", e=>{
      if(el.contains(e.target)) {
        this.$axios().then().finally();
      }
    })
  }
}
```

然后再在 main.js 之中全局引入

```javascript
// main.js

import Vue from "vue";
import router from './router';
import Monitor from "@/instruction/monitor";

Vue.directive("monitor", Monitor);

window.$vue = new Vue({
    router,
    render: h=>h(App)
}).$mounted("#app")
```

如此，大功告成。在此基础上对其进行的操作高效而又具有定向性，非常好使。

​	当然指令亦可注册为局部指令。这种情况一般是该指令权力较大，避免其他 `.vue` 文件滥用污染代码。注意的一点是局部指令只能在 `.vue` 中通过 `direvtives` 字段引入。

