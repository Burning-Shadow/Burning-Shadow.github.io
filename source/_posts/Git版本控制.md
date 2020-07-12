---
title: Git版本控制
date: 2020-07-12 16:52:03
tags:
 - 版本回退
 - 版本控制
 - git reset
 - git commit --amend
 - git rebase
---

对于多人协作项目 git 无疑是最重要的版本控制工具，而工作中一旦出现错误则可以通过版本回退来进行回滚，从而修复 BUG。

<!--more-->

### Repository 版本回退

#### 版本号

> 版本号是一个经由 SHA1 算法计算而来的40位 hash 串，用以保证自身的唯一性

版本号我们则可以通过 `git log` 在控制台中打印相应的版本信息。

```
$ git log
commit 1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master)
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:06:15 2018 +0800
    append GPL

commit e475afc93c209a690c39c13a46716e8fa000c366
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:03:36 2018 +0800
    add distributed

commit eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 20:59:18 2018 +0800
    wrote a readme file
```

而这种方式虽然包含的信息多但容易让人眼花，所以通过 `git log --pretty=oneline` 显然更令人舒服。

```
$ git log --pretty=oneline
1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master) append GPL
e475afc93c209a690c39c13a46716e8fa000c366 add distributed
eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0 wrote a readme file
```

#### HEAD

Git 拥有一个“头指针” HEAD，其指向当前所在工作区域对应的版本，而版本回退简单来说就是将我们的 HEAD 头指针移向之前的版本，工作原理类似于删除磁盘中的文件（其删除机理为将对应文件设置为“可覆盖”状态，并不物理清空相应区域，如有新内容写入直接覆盖即可）

#### CtrlZ

头指针指向的是当前版本，而上一个版本则是 `HEAD^`，上上个版本是 `HEAD^^`。当然也可以通过 `HEAD~frontVersion` 来表示。

但此种方式无疑风险过高，所以最好还是复制相应版本号。

```
git reset --hard HEAD^	// way 1
git reset --hard e475afc93c209a690c39c13a46716e8fa000c366   // way 2

HEAD is now at e475afc add distributed
```

而此时再 `git log` 则除去了删除的版本

```
$ git log
commit e475afc93c209a690c39c13a46716e8fa000c366 (HEAD -> master)
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:03:36 2018 +0800
    add distributed

commit eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 20:59:18 2018 +0800
    wrote a readme file
```

#### CtrlY

如果又想回到 **append GPL** 版本呢？

同样的命令

```
$ git reset --hard 1094adb7b9b3807259d8cb349e7df1d4d6477073

HEAD is now at 83b0afe append GPL
```

#### git reflog 补救

如果手滑关掉了电脑忘记了版本号怎么办？`git reflog` 会记录你的每一次命令。我们 `reset` 的时候版本号只复制前几位也是没问题的。

```
$ git reflog
e475afc HEAD@{1}: reset: moving to HEAD^
1094adb (HEAD -> master) HEAD@{2}: commit: append GPL
e475afc HEAD@{3}: commit: add distributed
eaadf4e HEAD@{4}: commit (initial): wrote a readme file
```

#### 备注

`git reset` 是一个非常好用的方法，当你完成 `reset` 操作后，那么被你撤回的操作又会被 Git 放在暂存区（Index），并且随时可撤回编辑，如此一来就不必担心版本回退后内容丢失的问题了。

### WorkSpace回退修改

如果代码仍处于 `Modified` 状态，那么我们除了通过上述方法进行版本回退之外还可以通过 `git checkout --<file>` 来撤销工作区相应文件的修改。

而其回退的版本则是你**最后修改版本之前**的内容，也就是说可能的情况有两种：

- `<file>` 自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
- `<file>` 已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态

> `git checkout --<file>` 的作用即是让文件回到最近一次 `git commit `或 `git add` 时的状态

### Index 回退修改

`git reset HEAD <file>` 可以将暂存区（Index）的修改撤掉（unstage），重新放回工作区（WorkSpace）

```
$ git reset HEAD readme.txt
Unstaged changes after reset:
M	readme.txt
```

### Remote前置钩子拦截

项目中远程仓库 Remote 往往会设置相应的钩子函数对 `push` 进行拦截，而其中 `commit` 则是比较重要的一环，保证 `commit` 备注信息的流程化和规范化可以让版本出现问题时可以更加高效的回滚和对问题进行追溯。

当 `commit` 内容不符合规范时，就需要用户重新填写相关的 `commit` 备注，而后悔药就是 `git commit --amend`

而其效果自然也是很明显 —— 更改最后一次的 `commit` 备注

（`i` 插入信息，修改完成后 Esc : wq。假设我们将备注由 `fix: 用户注册表失败,bug修复`  改为 `fix: 用户注册表失败,bug修复 --bug_id=80771171 修复注册故障`）

```
fix: 用户注册表失败,bug修复 --bug_id=80771171 修复注册故障
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Author:    qingyaochen <qingyaochen@tencent.com>
# Date:      Thu Jul 9 22:08:15 2020 +0800
#
# On branch bugfix/fate_schedule
# Changes to be committed:
#       modified:   src/pages/resource/components/RegisterData.vue
#
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
~                                                                                                  
"~/code/TC-webfront/.git/COMMIT_EDITMSG" 12L, 473C
```

此时我们再通过 `--graph` 展示一下变更图

```
* 6a107b9f (HEAD -> bugfix/fate_schedule) fix: 用户注册表失败,bug修复 --bug_id=80771171 修复注册故障
* 05c4b028 hotfix: 管理翻页参数错误修复，系统管理部分页面优化 --bug=80765149
* c41b5e9a fix 撤销和重做bug修复，校验画布是否存在重复边。--bug=80716183 撤回功能导致的连线改变未正
确修改xml (merge request !629)
*   8f858dfb Merge branch 'feature/workflow_fix' into 'develop' (merge request !627)
|\  
| *   e0a5e012 Merge branch 'develop' of http://git.code.oa.com/dp/jupiter-webfront into feature/wo
rkflow_fix
| |\  
| |/  
|/|   
* | 8be83f69 feat 周期化实例运行 --story_id=858436119 【TC周常优化】日常积累问题优化——7w1
* | cfe04e62 fix: 数据模块功能优化 --story_id=859018101 新增我的收藏，我的保存位置迁移 (merge request !626)
* | c19b360c feature/model_time (merge request !625)
* |   8cf00538 Merge branch 'feature/workflow_fix' into 'develop' (merge request !623)
|\ \  
* \ \   449be090 fix: TC单元测试增加 --story_id=858832859 补充公共方法单元测试，增加storybook组件ui管理工具 (merge request !624)
|\ \ \  
| * | | e49dd58d fix: TC单元测试增加 --story_id=858832859 工具方法单元测试问题修复
| * | | 1a2c2c8f fix: TC单元测试增加 --story_id=858832859 代码优化
| * | | c37730a6 fix: TC单元测试增加 --story_id=858832859 删除未上线功能
| * | | 38dd5d00 fix: TC单元测试增加 --story_id=858832859 补充公共方法单元测试，增加storybook组件ui管理工具
* | | | 193db219 feat: 合并作业状映射 --story_id=858968467 作业状态映射
|/ / /  
| | * 18d7357f feat 调整上报代码。--story_id=859021431 TC-上报代码优化
```

如此一来我们的备注信息就会变化啦，再执行 `push` 操作就能完成 Remote 端前置钩子的检验啦~

### Rebase 变基

廖雪峰老师的教程中 Rebase 主要用来对 graph 进行更改。使得杂乱的提交图更加有序。