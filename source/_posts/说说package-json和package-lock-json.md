---
title: 说说package.json和package.lock.json
date: 2019-11-17 20:10:24
tags:
 - package.json
 - package.lock.json
---

​		在迁移文件的时候我们往往无需拷贝 `node_modules` 文件夹，而是对 `package.json` 进行拷贝，完成后则进行 `npm install` 即可完成依赖项的安装。那么 `package.lock.json` 呢？

<!--more-->

### 版本

​		我们在下载 npm 社区所提供的包时往往直接 `install`。但是在具体的项目中就不太合适了。因为每一个应用都有其对应的版本号所绑定，一旦涉及到版本更新那么也许会出现不兼容的情况。而版本也分为三部分：X、Y、Z

- X：指代大版本。往往更新大版本时会带来新的使用方面的问题，需要进行调整
- Y：指小版本。一般来说是增加新功能，并不影响使用。
- Z：`bugfix` 版本，不会影响任何功能。

​	而 npm 在完成安装后则会在 `package.json` 文件中写入其对应信息。比如我们执行 `npm install express --save` 后其自动安装了 `5.7.2` 版本，此时会在 `package.json` 中添加一行 `"express": "^5.7.2"`。那个 `^` 表示最低版本不得低于 `5.7.2`。

​		而在我们重新进行安装时只要保证大版本（X）相同的情况下，npm 会为我们**默认安装最新的包**。

### package.json

​		`npm` 是 `node package manager` 的缩写，用于管理各个包之间的依赖关系。 

举个例子，在 `package.json` 中会出现诸如此类的标记

```json
{
  "name": "hexo-site",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": "3.8.0"
  },
  "dependencies": {
    "hexo": "^3.8.0",
    "hexo-deployer-git": "^1.0.0",
    "hexo-generator-archive": "^0.1.5",
    "hexo-generator-category": "^0.1.3",
    "hexo-generator-index": "^0.2.1",
    "hexo-generator-tag": "^0.2.0",
    "hexo-renderer-ejs": "^0.3.1",
    "hexo-renderer-marked": "^0.3.2",
    "hexo-renderer-stylus": "^0.3.3",
    "hexo-server": "^0.3.3"
  }
}
```

​		前面我们讲了其版本部分的特征 —— 安装时在保证大版本不变的情况下默认安装最新版本。

​		一般情况这种向下兼容最新版本并不会出现问题，**但也不一定**。

> 在完全相同的一个 nodejs 的代码库，在不同时间或者不同 npm 下载源之下，下到的各依赖库包版本可能有所不同，因此其依赖库包行为特征也不同有时候甚至完全不兼容。 

​		原本的解决之道是 shrinkwrap。但是另一方面，`yarn.lock` 也着实给 `npm` 带来不小的压力。所以，我们的 `package.lock.json` 应运而生

### package.lock.json

​		由于 `package.json` 只能锁定大版本，而后面的小版本，则是需要我们的 `package.lock.json` 来进行**锁定**。如此一来就解决了我们在大版本进行拉取时的兼容性问题，以减少我们的测试/适配的工作量。

​		npm 在 `5.0` 版本之后默认都会为我们加上 `package.lock.json` ，所以大家伙儿不需要担心。

​		但是！`npm 5.0` 发布以来，`npm install` 时的规则发生了三次变化

> - `npm 5.0.x` 版本，不管 `package.json` 怎么变，`npm i` 时都会根据lock文件下载 [package-lock.json file not updated after package.json file is changed · Issue #16866 · npm/npm](https://link.zhihu.com/?target=https%3A//github.com/npm/npm/issues/16866) 这个 issue 控诉了这个问题，明明手动改了 `package.json`，为啥不给我升级包！然后就导致了 `5.1.0` 的问题...
> - `5.1.0` 版本后 `npm install` 会无视 `package.lock.json` 文件，而去下载最新的包 。然后有人提了这个 `issue`:  [why is package-lock being ignored? · Issue #17979 · npm/npm](https://link.zhihu.com/?target=https%3A//github.com/npm/npm/issues/17979) 控诉这个问题，最后演变成 `5.4.2` 版本后的规则。
> - `5.4.2` 版本后大致规则是，如果改了 `package.json`，且 `package.json` 和 `lock` 文件不同，那么执行 `npm i` 时 `npm` 会根据 `package.json` 中的版本号以及语义含义去下载最新的包，**并更新至 `lock`**。如果两者是同一状态，那么执行 `npm i ` 都会根据 `lock` 下载，不会理会 `package` 实际包的版本是否有新。

​		最终，在经历了三个小版本的迭代后社区中统一了 `package.json` 和 `package.lock.json` 的权重问题，使得更新 `package.json` 文件时二者的同步问题得以解决。