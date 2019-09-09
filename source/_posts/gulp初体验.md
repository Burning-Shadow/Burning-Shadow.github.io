---
title: gulp初体验
categories: 打包工具
tags:
 - gulp
---

`gulp`相较于`webpack`更好学一点，而且由于其轻量化和异步等特点更适合我这种菜鸡入门，所以先学`gulp`再看`webpack`叭~

<!--more-->

### 咋整

首先我们需要的是建立一个`.js`文件。名字你随便取，但是里边的内容还是有规定的。大致分为三部分

```javascript
var gulp = require('gulp');

// 注册任务
gulp.task("任务名", function(){
    // 配置任务的操作
})

// 注册默认任务
gulp.task("default", ["任务名1", "任务名2", ....])
```

之后控制台输入`gulp`，回车就会自动执行`gulp.task`中的任务咯

#### 相关插件

- `gulp-concat`：合并文件（js/css）
- `gulp-uglify`：压缩 js 文件
- `gulp-rename`：文件重命名
- `gulp-less`：编译 less
- `gulp-clean-css`：压缩 css
- `gulp-livereload`：实时自动编译刷新

#### 重要 API

- `gulp.src(filePath/pathArr)`
  - 指向指定路径的所有文件，返回文件流对象
  - 用于读取文件
- `gulp.dest(dirPath/pathArr)`
  - 指向指定的所有文件
  - 用于向文件夹中输出文件
- `gulp.task(name, [deps], fn)`
  - 定义一个任务
- `gulp.watch()`
  - 监视文件的变化

### 合并压缩js

一般来讲 js 是构建是大家伙儿用得比较多的。所以我们先从这里入手

```javascript
var gulp = require("gulp");
var concat = require("gulp-concat");
var uglify = require("gulp-uglify");
var rename = require("gulp-rename");
var less = require("gulp-less");
var cleanCss = require("gulp-clean-css");
var htmlMin = require("gulp-htmlmin");


gulp.task("js", function(){
    return gulp.src("src/js/**/*.js")	// 将源文件中的数据读取到 gulp 内存中
    .pipe(concat("build.js"))		// 将我们的两个目标js文件临时合并
    .pipe(gulp.dest("dist/js/"))	// 将 build.js 输出到 dist 目录下的 js 文件中
    .pipe(uglify())				// 压缩文件
    // .pipe(rename("build.min.js"))	// 重命名
    .pipe(rename({ suffix: ".min" }))	// 重命名方法2
    .pipe(gulp.dest("dist/js/"))	// 将build.min.js 输出到 dist 目录下的 js 文件中
})
```

fine，我们先大致讲解一下。

`gulp.task`显然是注册一个压缩`js`的任务。第一个参数为被压缩文件的类型，其后则是我们的构建函数。

其中我们注意到这句的写法：`return gulp.src("src/js/**/*js")`

那两个`*`的意思是进行一个深度遍历。将 js 文件下的所有`.js`文件全部读取。若去掉则代表浅度遍历。

而我们中间进行的那一步`gulp.dest("dist/js/")`没有意义，只是想告诉大家伙儿我们什么时候输出都可以。

> 值得注意的一点，若我们在代码中定义的变量未使用则在打包完成后的 `.min.js`文件中不予显示。甚至变量定义都被裁掉了。这也进一步的减少了占用空间。

随后我们在控制台输入`gulp + 任务名`，在本例中也就是`gulp js`，即可完成压缩

### 合并压缩CSS

#### 转化less

很多 boy 喜欢使用这些预编译插件，让我们可以更好的看清逻辑。所以我们先对`.less`文件进行转换，使其变为`.css`。在此之前当然少不了引包了。因为懒我上边一次性引好了。

```javascript
var gulp = require("gulp");
var less = require("gulp-less");

gulp.task("less", function(){
    return gulp.src("src/less/**/*.less")
    .pipe(less())
    .pipe(gulp.dest("dist/css/"))
})
```

随后我们在控制台输入`gulp + 任务名`，在本例中也就是`gulp less`，即可完成编译

这里注意我的操作。最后将文件输出到了`dist`目录下的`css`文件夹中，以方便我们今后对`.css`文件进行编译

#### 合并&压缩css

```javascript
var gulp = require("gulp");
var concat = require("gulp-concat");
var uglify = require("gulp-uglify");
var rename = require("gulp-rename");
var less = require("gulp-less");
var cleanCss = require("gulp-clean-css");
var htmlMin = require("gulp-htmlmin");

gulp.task("css", function(){
    return gulp.src("src/css/**/*.css")
    .pipe(concat("build.css"))	// 合并多个css文件，合成文件名字为build.css
    .pipe(uglify())
    .pipe(rename({ suffix: ".min" }))
    .pipe(cleanCss({ compatibility: "ie8" }))	// 兼容 ie8
    .pipe(gulp.dest("dist/css/"))
})
```

控制台输入`gulp css`， css 文件就完成压缩和合并啦 

OK 我们大功告成。这样就会把 CSS 文件全部压缩为一个`build.min.css`啦

#### 真的大功告成了？

诸位兄弟有没有注意到我们的**`.less`文件是依赖于`.css`文件的**？而且`gulp`最大的优势还是**异步编译**！咱这得有个先后顺序呀！所以我们进行 css 编译的时候需要加一个参数：一个`Array`！

```javascript
gulp.task("css", ["less"], function(){
    ....
})
```

里边的`["less"]`是我们的依赖项。在完成`.less`文件编译之前是不会进行`.css`文件的编译的。这就完美的解决了我们的**异步执行时任务之间的依赖问题**。触类旁通，其他的也一样哈。

### 构建HTML

这玩意儿可不叫压缩了，HTML 代码好像没法怎么压缩，也只能删删行与行之间的空格维持一下生活的样子

```javascript
var gulp = require("gulp");
var concat = require("gulp-concat");
var uglify = require("gulp-uglify");
var rename = require("gulp-rename");
var less = require("gulp-less");
var cleanCss = require("gulp-clean-css");
var htmlMin = require("gulp-htmlmin");
var htmlMin = require("gulp-htmlmin");

gulp.task("html", function(){
    return gulp.src("index.html")
    .pipe(htmlMin({ collapseWhitespace: true }))	// 合并空白
    .pipe(gulp.dest("dist/"))
})

gulp.task("default", ["js", "less", "css", "html"])	// 注册默认任务
```

控制台输入`gulp html`， html 文件就完成压缩和合并啦 

大致就是这亚子，html 这里好像真的没什么可说的。唯一要注意的一点就是**注意路径**！咱们打包之后的相对路径一定要和打包之前的相对路径一样，否则你会发现你的 css 和 js 文件都离你而去，剩一个光秃秃的 html，特别尴尬。

### 半自动项目构建

咱们有的时候非得动一下某个参数，然后老规矩，编译，完了再重新刷新一下网页才能看到效果。`gulp`是有这样半自动化的构建工具的：`npm install gulp-livereload --save-dev`

```javascript
var livereload = require("gulp-livereload");

// ...注册压缩html、css、js等等的任务....
gulp.task("html", function(){
	....
    .pipe(livereload())	// 实时刷新
});
gulp.task("less", function(){
	....
    .pipe(livereload())	// 实时刷新
});
gulp.task("css", function(){
	....
    .pipe(livereload())	// 实时刷新
});
gulp.task("js", function(){
	....
    .pipe(livereload())	// 实时刷新
});
          

// 注册监视任务
gulp.task("watch", ["default"], function(){
    livereload.listen();	// 开启监听
    gulp.watch("src/js/*.js", ["js"]);	// 确定监听的目标及绑定相应的任务
    gulp.watch(["src/css/*.css", "src/less/*.less"], ["css"]);	// 由于我们css任务的依赖是less任务，所以我们只需要启动css任务即可完成对less任务的运行
})

gulp.task("default", ["js", "less", "css", "html"])
```

随后执行`gulp watch`。你就理解为热部署好啦~。如此你更改某项依赖文件则可以实时的反映在你的页面之中。

### 全自动？

刚才那个我觉着挺屌了但只能算是半自动。那怎么算是全自动？咱们用 vue 写项目的时候大多数好兄弟用的都是`vue-cli`叭。有没有注意到我们更该`.vue`文件之后他随后就会实时的更新到页面上，这个过程都不需要我们刷新页面！

咱们所说的全自动也可以完成！

插件下载`npm install gulp-connect --save-dev`

```javascript
var connect = require("gulp-connect");

// ...注册压缩html、css、js等等的任务....下边加一个connnect.reload()任务保证实时刷新
gulp.task("html", function(){
	....
    .pipe(livereload())	// 实时刷新
    .pipe(connect.reload())
});
gulp.task("less", function(){
	....
    .pipe(livereload())	// 实时刷新
    .pipe(connect.reload())
});
gulp.task("css", function(){
	....
    .pipe(livereload())	// 实时刷新
    .pipe(connect.reload())
});
gulp.task("js", function(){
	....
    .pipe(livereload())	// 实时刷新
    .pipe(connect.reload())
});

gulp.task("server", ["default"], function(){
    // 配置服务器的选项
    connect.server({
        root: "dist/",
        livereload: true,
        port: 8080
    });
    
    // 确认监听的目标以及绑定相应的任务
    gulp.watch("src/js/*.js", ["js"]);	// 确定监听的目标及绑定相应的任务
    gulp.watch(["src/css/*.css", "src/less/*.less"], ["css"]);	// 由于我们css任务的依赖是less任务，所以我们只需要启动css任务即可完成对less任务的运行
})
```

`gulp-connect`插件内部有一个微型服务器，用于读取当前配置文件的所有配置，并将你所需要加载的依赖、执行的任务以及全部的源文件。并将其全部读取到其微型服务器中，传入一个地址他会帮你默认在此地址中打开。

如此，我们只需要在控制台运行命令`gulp server`即可完成部署

### 还有插件？

咱们完成之后让他可以自动打开网页。有点类似于`venus`平台。

加载完成后自动打开指定的链接：`npm install open --save-dev`

```javascript
var open = require("open")

// 中间的和上边一样

open("http://localhost:8080")
```

### gulp-load-plugins

这玩意儿能干嘛？

~~不能~~  它是用来**提前帮你打包基于`gulp`的所有插件**。

```javascript
var gulp = require("gulp");
var $ = require("gulp-load-plugins");

// 引入完成之后，由于$是被传回的第一个对象，所以我们在 .pipe() 管道中调用各种方法前面再加个 "$." 
// 当然，我们上一句的适用范围是各种非 gulp 的原始包。而比如 gulp.dest() 方法前边就不加啦
// 还有一个例外就是open。因为他是帮你打包 基于gulp 的所有插件！基于!
// 好记一点就是：凡是gulp开头的方法都不用加
// 这样我们又可以少写不少代码咯
```

> 不过咱哥几个需要注意的一点是，我们在此基础上不能换数据！他会自动把中间的"-"连字符命名法转换成驼峰命名法。所以起名的时候慎重点看清楚咯。