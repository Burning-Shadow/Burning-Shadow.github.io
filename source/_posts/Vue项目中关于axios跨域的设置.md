---
title: Vue项目中关于axios跨域的设置
categories: Vue
---

`config`文件夹中的`dev.env.js`

```javascript
proxyTable: {
    '/api': {
        target: 'http://192.168.1.1:80', // api代理服务器地址
        changeOrigin: true, // 允许跨域
        secure: false,
        pathRewrite: {
            '^/api': '/'
        }
    }
}
```

`target`是我们要将请求发送的靶向目标。此选项可以理解为我们的`node`后台为我们作了一次正向代理，以此避免了跨域。

<!--more-->

`config`文件夹中的`index.js`

```javascript
const devEnv = require('./dev.env')
module.exports = {
    dev: {
        // Paths
        assetsSubDirectory: 'static',
        assetsPublicPath: '/',
        proxyTable: devEnv.proxyTable,
        
        //....
    }
}
```

如此我们每次只需要更改`dev.env.js`文件夹中的`target`项即可。



使用`axios`跨域时候我们需要在`main.js`中声明

```javascript
import Axios from 'axios'
Vue.prototype.$axios = Axios
```

发送请求时别忘了`$`

```javascript
this.$axios({
    // ....
})
```

