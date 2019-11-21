---
title: 说说package.json和package.lock.json
date: 2019-11-17 20:10:24
tags:
 - package.json
 - package.lock.json
---

​		在迁移文件的时候我们往往无需拷贝 `node_modules` 文件夹，而是对 `package.json` 进行拷贝，完成后则进行 `npm install` 即可完成依赖项的安装。那么 `package.lock.json` 呢？

<!--more-->

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

​		而前面的 `^` 代表的就是 **向后兼容依赖**。比如 `"hexo": "^3.8.0"` 表示的就是，若 `hexo` 版本超过 `3.8.0`，且在大版本号 (3) 上相同，即允许下载**最新版本**的包。

​		一般情况这种向下兼容最新版本并不会出现问题（大的改动往往是大版本进行更新时），但也不一定。

> 在完全相同的一个 nodejs 的代码库，在不同时间或者不同 npm 下载源之下，下到的各依赖库包版本可能有所不同，因此其依赖库包行为特征也不同有时候甚至完全不兼容。 

​		原本的解决之道是 shrinkwrap。但是另一方面，`yarn.lock` 也着实给 `npm` 带来不小的压力。所以，我们的 `package.lock.json` 应运而生

### package.lock.json

​		由于 `package.json` 只能锁定大版本，而后面的小版本，则是需要我们的 `package.lock.json` 来进行**锁定**。如此一来就解决了我们在大版本进行拉取时的兼容性问题（一般来讲是不敢随便 `npm install` 的，因为它会自动拉取大版本下的最新版安装包），以减少我们的测试/适配的工作量。

​		但是！`npm 5.0` 发布以来，`npm install` 时的规则发生了三次变化

> - npm 5.0.x 版本，不管package.json怎么变，npm i 时都会根据lock文件下载 [package-lock.json file not updated after package.json file is changed · Issue #16866 · npm/npm](https://link.zhihu.com/?target=https%3A//github.com/npm/npm/issues/16866) 这个 issue 控诉了这个问题，明明手动改了package.json，为啥不给我升级包！然后就导致了5.1.0的问题...
> - 5.1.0版本后 npm install 会无视 lock 文件 去下载最新的 npm 。然后有人提了这个 issue  [why is package-lock being ignored? · Issue #17979 · npm/npm](https://link.zhihu.com/?target=https%3A//github.com/npm/npm/issues/17979) 控诉这个问题，最后演变成 5.4.2 版本后的规则。
> - 5.4.2 版本后  [why is package-lock being ignored? · Issue #17979 · npm/npm](https://link.zhihu.com/?target=https%3A//github.com/npm/npm/issues/17979) 大致意思是，如果改了 package.json，且 package.json 和lock文件不同，那么执行 `npm i` 时 npm 会根据 package 中的版本号以及语义含义去下载最新的包，并更新至 lock。如果两者是同一状态，那么执行 `npm i ` 都会根据 lock 下载，不会理会 package 实际包的版本是否有新。

