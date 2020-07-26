---
title: Git版本控制
categories: Git
tags:
 - git
 - 版本控制
 - 版本回退
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

### Rebase 

廖雪峰老师的教程中 Rebase 主要用来对 graph 进行更改。使得杂乱的提交图更加有序。举一个尤雨溪大大的 vue 项目提交记录图，可以看到其提交基线很是工整









而变基的原理则是对本地的内容顺序进行更改。

> 我们可以将整个分支体系想象成一条单向链表，将次分支提交的变更提取，存为临时文件，然后将变更依附于当前 `master` 分支的新 `commit`，使用 `git base <branch-name>`，使用次分支的 `.next` 指向最新的 `master` 分支。最后再将 `master` 分支与次分支 `merge` ，以此来达到基线不打结的目的。

#### Tips

> 除此之外，`pull` 后若代码有冲突，先不要着急解决冲突，因为修改后执行 `rebase` 操作还是会变成冲突前的代码。
> 先 `git add .` 和 `git commit -m "xxxx"` 。随后执行 `git rebase` 时终端会显示
>
> ```shell
> Resolve all conflicts manually, mark them as resolved with "git add/rm <conflicted_files>", then run "git rebase --continue
> ```
>
> 此时再手动修改代码解决冲突，执行 `git add .` 再执行 `git rebase --continue` 就有效果了。完成后直接 `push` 即可。
>
> **因为 `pull` 下来的冲突和 `rebase` 后的冲突是不一样的，前者是远程库版本对比所导致的冲突，而后者则是变基后和版本对比所导致的冲突**

而我们一旦出现错误，诸如希望更改未提交的 `commit` 却手抖将远程代码 `pull` 了下来，此时就需要我们的 `rebase` 操作啦

#### 通过 rebase 合并多条请求

通过 `git rebase -i <commit-code>` 进入交互模式（`interactive`） ，而 `commit-code` 则是你希望修改的 `commit` 之前的 `commit`

完成后会进入编辑页面，显示着如下几行文本（#开头的是注释，此处则是代表具体情况下的格式，可忽略）

```shell
pick deadbee The oneline of this commit
pick fa1afe1 The oneline of the next commit
# pick 4fcc9143 add: add some new tips
...
```

其中每一行代表一个 `commit` ，而 `pick` 则表示要对该 `commit` 做的操作。而我们接下来要做的就是对每行前面所代表的命令的词汇进行修改。共有 5 种操作： edit、reword、drop、squash、fixup

##### edit

`edit` 命令表示你告诉了 `rebase`，当在应用这个 `commit` 的时候，停下来，等待你修改了文件 和/或 修改了`commit messag` 之后在继续进行 `rebase`。简而言之就是既可修改 `commit message` 又可修改相应 `commit` 的文件内容

##### reword

`reword` 命令可以让你修改 `commit message`。当你使用这个命令后，保存这个文件并退出，执行 `git rebase continue` 命令之后会再次打开一个文件，让你对这个 `commit` 的 `commit message` 进行修改，再次保存退出之后继续进行 `rebase`

##### drop

`drop` 命令表示你要丢弃这个 `commit` 以及它的修改。同样可以删除这一行来表示。

##### squash & fixup

这两个命令都是用来将几个 `commit` 合并为一个的。其中，`fixup` 命令 `rebase` 的时候将会直接忽略掉它的 `commit message`，而 `squash` 命令，则会在 `git rebase --continue` 之后打开一个文件，该文件中将会出现所有设置为 `squash` 的 `commit`，这时删除掉多余的 `commit message`，留下（或者修改）一行作为合并之后的 `commit` 的 `commit message`。



