---
title: Webpack初体验
categories: 打包工具
tags
 - webpack
---

​	`webpack` 作为一个现代的 JS 应用程序的静态打包工具(`module bundler`) 是没一个前端都绕不开的必啃项目。为了防止遗忘所以在学习的过程中我们做一些相应的笔记，以此来完善知识体系

<!--more-->

### 提要

> `webpack`：静态解析。项目完成后从入口开始解析，将依赖项打包，非依赖项不进行处理
>
> `glup`：按照某种规则对文件进行匹配，成功则执行对应流程，一个一个进行处理
>
> `mode`：开发模式 `production`
>
> `entry`：入口文件
>
> `output`：最终打包完成的`bundle`放至何处
>
> `loader`：用`loader`处理非 JS 文件，将其转化为`webpack`所能处理的有效模块（JS、JSX）
>
> `plugins`：打包优化、压缩、重新定义环境中的变量等等。。。

### 加载非JS文件

​	由于`webpack`无法解析非 JS 文件，所以我们需要对其进行预处理，将其解析为`webpack`可理解的形式。

#### CSS文件

我们需要两个`loader`：`css-loader`和`style-loader`

```
npm i -D style-loader css-loader
```

此后就需要更改我们的配置项文件(`webpack.config.js`)啦

```javascript
const path = require('path');

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist")
  },
  module: {
    rules: [{
      test: /\.css$/,
      use: ["style-loader", "css-loader"] // 处理过程从右向左，先执行 css-loader 再执行 style-loader
    }]
  }
};
```





### 初始工具

```
mkdir webpack-demo && cd webpack-demo
npm init -y
npm install webpack webpack-cli --save-dev
```

`webpack-cli`用于再命令行中运行`webpack`。

一般来说我们会选择局部而非全局安装`webpack`，这样会有固定的版本号，以至于我们更新时不会将某些依赖项改变





### 使用教程

#### webpack.config.js中的配置项

​	`webpack.config.js`中一般是用来存储配置项

```javascript
const path = require('path');

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist")
  },
  module: {
    rules: [{
      test: /\.css$/,
      use: ["style-loader", "css-loader"] // 处理过程从右向左，先执行 css-loader 再执行 style-loader
    }]
  }
};
```

​	此例中我们通过`use`字段规定了处理以`.css`结尾的文件的处理方式。由于`webpack`无法解析除去 JS、JSX语法之外的其他语言，所以在此我们需要使用`css-loader`将其解析为`webpack`可处理的语言，再交由`style-loader`将其渲染至页面上。

​	而具体引入方法就是将该`css`文件引入至将要引入页面的 JS 文件之中去

##### `module`中其他配置项

- `module.noParse`：此类型为了防止`webpack`解析那些与正则相匹配的文件。举个栗子，我们不需要对`jquery`、`lodash`等文件进行解析

  - ```javascript
    const path = require('path');
    
    module.exports = {
      entry: "./src/index.js",
      output: {
        filename: "bundle.js",
        path: path.resolve(__dirname, "dist")
      },
      module: {
        noParse: /jquery|lodash/
        // 写成函数样式的也可以
        // noParse: function(content){
        //  return /jquery|lodash/.test(content);
        //}
      }
    };
    ```

  - 当然我们也可以采用另一种写法，毕竟

- `module.rules`：这个的话上边的例子有，参照那个就好啦

- `module.rule`：`rules`中的具体配置项。其`test`项既可以是字符串（目录绝对路径或是文件绝对路径）、正则表达式、函数（返回为`true`的就可以匹配），亦可以是对象（匹配所有属性。每一个属性都有一个定义行为）和条件数组（至少一个匹配条件）

  - `Rule.test`：既可以为一个正则、函数，亦可以是一个正则数组或条件数组
  - `Rule.include`：匹配特定条件
  - `Rule.exclude`：排除特定条件
  - `Rule.and`：必须匹配数组中的所有条件
  - `Rule.or`：匹配数组中的任一条件
  - `Rule.not`：必须排除某条件
  - `Rule.use`：应用于模块指定使用一个`loader`

我们举个例子。由于`node_module`文件夹中一般来讲发布到线上后会从 CDN 上获取资源，所以不需要对其进行打包，自然也是忽略掉。折至`webpack.config.js`中的配置为

```javascript
module.exports = {
  ....
  module: {
    rules: [
      {
        test: [/\.css$/, /\.js$/, /\.html$/, /\.ejs$/],
        exclude: [
          path.resolve(__dirname, "node_modules")
        ],
        use: ["style-loader", "css-loader"]
      }
    ]
  }
};
```

#### Sourse Map

经过打包后的文件都被封装过，而希望找到其最初始的位置我们需要`Source Map`

比如，在`css-loader`和`sass-loader`中我们都可以通过`options`配置项启用`sourcemap`

```javascript
module.exports = {
  ....
  module: {
    rules: [
      {
        test: /\.(sc|sa|c)ss$/,
        use: [{
    	  loader: "style-loader"
		}, {
    	  loader: "css-loader",
    	  options: {
            sourceMap: true
          }
		}, {
    	  loader: "sass-loader",
    	  options: {
            sourceMap: true
          }
		}]
      }
    ]
  }
};
```

#### PostCSS

是一个 CSS 预处理插件，可以帮助我们：

- 给 CSS3 的属性添加前缀
- 样式格式校验（`stylelint`）
- 实现 CSS 模块化，防止 CSS 样式冲突

```
npm i -D postcss-loader autoprefixer
```

举个栗子，我们上方在完成`.sass`的解析后在让`css-loader`进行解析之前需要对一些 CSS3 特性加上前缀。那么就在 `Rule.use` 项中添加一个配置项

```javascript
{
  loader: "postcss-loader",
  options: {
    ident: "postcss",
    sourceMap: true,
    plugins: loader => {
      require("autoprefixer")({ browsers: {'> 0.15% in CN' } })	//添加前缀
    }
  }
}
```

