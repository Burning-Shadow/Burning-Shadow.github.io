---
title: yarn初体验
date: 2019-05-21 21:46:00
categories: Node Package Manager
tags:
 - yarn
---

作为更快更高更强的包管理工具`yarn`的出现无疑赚走了`npm `的一部分市场。今天我们暂不究其原理，只刷文档，能用就行。

<!--more-->

### 创建一个新项目

执行命令：`yarn init`

其后将打开一个用于创建`yarn`项目的交互式表单

```
name(your-project):
version(1.1.0):
description:
entry point(index.js):
git repository:
author:
license(MIT):
```

跟你用`npm init`一样，一般咱们这不讲究的就全回车默认配置了。

完成后他会创建一个和下面文件内容类似的`package.json`

```
{
	"name": "my_new_project",
	"version": "1.0.0",
	"description": "My New Project description",
	"main": "index.js",
	"repository": {
		"url": "https://example.com/your-username/my-new-project",
		"type": "git"
	},
	"author": "Your Name <you@example.com>",
	"license": "MIT"
}
```

### 管理依赖项目

#### 添加依赖包

众所周知使用`npm`时是通过`npm install [package]`来完成依赖包的安装。而`yarn`则是通过`yarn add [package]`来对依赖进行装载。效果同`npm `那句相同，`[package]`都会被加入到`package.json`的依赖列表中，同时`yarn.lock`也会被更新。



当然，我们还可以通过参数添加其他类型的依赖

```
yarn add --dev       添加到devDependencies
yarn add --peer      添加到peerDependencies
yarn add --optional  添加到optionalDependencies
```

而版本号也一样，`yarn add package@1.1.0`即可

#### 更新依赖包

```
yarn upgrade [package]
yarn upgrade [package]@[verision]
yarn upgrade [package]@[tag]
```

这会更新`package.json`和`yarn.lock`

```
{
	"name": "my-package",
	"dependencies": {
		"package-1": "^1.0.0",
		"package-2": "^2.2.0"
	}
}
```

#### 移除依赖包

```
yarn remove [package]
```

### 安装依赖项

我们可以通过`yarn install`来安装一个项目中所有的依赖项。Yarn 会自动从`package.json`中读取依赖，并将依赖信息存储到`yarn.lock`中

- 安装所有依赖：`yarn install`
- 安装一个包的单一版本：`yarn install --flat`
- 强制重新下载所有包：`yarn install --force`
- 只安装生产环境依赖：`yarn install --production`

### 配合版本控制

- `yarn.lock`用于记录每一个依赖项的确切版本信息（作用同`package.lock.json`）

- `package.json`则包函包的所有依赖信息

### 说道说道yarn.lock和package-lock.json

先说结论

>`package-lock.json`是 npm5 之后默认生成的锁文件，而`yarn.lock`是 yarn 的锁文件
>
>`yarn.lock`就好比是你的`package-lock.json`，不同的是`package-lock.json`有不完全扁平化的问题，而`yarn.lock`则把所有依赖包扁平化的展示了出来（同包名但`semver`不兼容的作为不同的字段放在了`yarn.lock`的同一级结构中）













