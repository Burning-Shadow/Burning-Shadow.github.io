---
title: Git常用操作
date: 2019-07-04 16:38:00
categories: Git
tags:
 - 分支合并
 - 入栈出栈
---

在公司中`git`是每个程序员都需要掌握的技能，今天就在此将一些常用的命令做一个总结以备日后所需

<!--more-->

### 常用操作合集

#### 日常操作

`$ git st(status)`   # → 查看当前分支工作区、暂存区的工作状态

`$ git diff`    # → diff文件的修改（⚠️很重要很重要很重要） 

`$ git ci(commit)`     # → 提交本次修改

`$ git fetch --all`    # → 拉取所有远端的最新代码 

`$ git merge origin/develop`    # → 如果是多人协作，merge同事的修改到当前分支（先人后己原则）

`$ git merge origin/master`    # → 上线之前保证当前分支不落后于远端origin/master，一定要merge远端origin/master到当前分支 

`$ git push`    # → 推送当前分支到远端仓库 

#### 权限操作

**创建开发分支`develop`**：`$ git co(checkout) -b develop`

**合并分支**：`$ git merge --no-ff origin/develop`    # → 同事`review code`之后管理员合并`origin/develop`到远端主干`origin/master`


### 冲突合并

项目中由于大家都是独立开发，完成后再将自己负责的部分推到线上。而某些时候，比如你和另外的老哥同时更改了同一个文件，此时后拉下来代码的人就需要将分支进行合并咯~（所以勤推代码是好事儿）

#### 是什么

在 Git 中，“合并（merging）” 是在形式上整合别的分支到你当前的工作分支的操作。你需要得到在另外一个上下文背景下的改动（这就也就是我们所提到过的，一个有效的分支应该是建立在一个上下文工作背景上的），并且合并它们到你的当前的工作文件中来。

（这段是我粘贴来的，我自己也没搞懂，就是吓唬你们一下

#### 咋整

我们当面对一个被修改的文件时，可能有以下几种情况：

- 部分增加
- 部分删除
- 部分修改（有增有删）

当咱使用 “`git status`” 时， Git 会告诉你存在一个 “未合并的路径（`unmerged paths`）”，这只是用另外一个方式告诉你，存在一个或多个冲突：

```
$ git status
# On branch contact-form
# You have unmerged paths.
#   (fix conflicts and run "git commit")
#
# Unmerged paths:
#   (use "git add <file>..." to mark resolution)
#
#       both modified:   contact.html
#
no changes added to commit (use "git add" and/or "git commit -a")
```

当改动发生在同一个文件的同一行上，我们就需要看看发生冲突的文件的内容

Git 会将文件中有问题的区域在“`<<<<<<< HEAD`” 和 “`>>>>>>> [other/branch/name]`” 之间标记出来。

![问题区域](https://ae01.alicdn.com/kf/HTB1y4iRe.KF3KVjSZFE760ExFXaq.png)

##### 强大的编辑器

如果使用编辑器（`idle`、`vscode`等）会有默认的自动提示，我们只需要在对应区域选择 “保留原版代码 / 保留现有代码 / 保留双方代码”即可。（如果有冲突则保留双方代码无法直接使用），但是命令行的话就需要我们手动来做更改了。

##### 命令行方式

我们经常通过`git status`来查看在本地副本（working copy）有哪些文件被改动了，而若想清楚的了解这些改动的细节，就需使用`git diff`

```
$ git diff
diff --git a/about.html b/about.html
index d09ab79..0c20c33 100644
--- a/about.html
+++ b/about.html
@@ -19,7 +19,7 @@
   </div>

   <div id="headerContainer">
-    <h1>About&lt/h1>
+    <h1>About This Project&lt/h1>
   </div>

   <div id="contentContainer">
diff --git a/imprint.html b/imprint.html
index 1932d95..d34d56a 100644
--- a/imprint.html
+++ b/imprint.html
@@ -19,7 +19,7 @@
   </div>

   <div id="headerContainer">
-    <h1>Imprint&lt/h1>
+    <h1>Imprint / Disclaimer&lt/h1>
   </div>

   <div id="contentContainer">
```

![读懂git文件](https://ae01.alicdn.com/kf/HTB1Qq6keW1s3KVjSZFAq6x_ZXXaL.jpg)

- 元数据（`Metadata`）：最开始的两串数字表示两个文件的 hashes（简单点说就是它们的 “ID”）。这个 hash ID 就代表了一个文件对象的特定版本。最后的一串数字代表了一个文件的模式（100644 代表它是一个普通的文件，100755 表示一个可执行文件，120000 仅仅是一符号链接）。**开发中版本发布时最好带上文件的元数据，这样方便查找和版本回退。**
- 标记（a/b）：继续向下观察这些输出信息，A 与 B 的真正差别会被显示在这里。新增部分（新版本）会使用"+"表示，而老版本则是"-"

若不带参数则`git diff`会为我们给所有**在本地副本中且未被打包（`unstaged`）的变化**作比较并展示

而若仅仅想看那些已经被打包的改动结果则可以使用`git diff --staged`

进入编辑器中手动合并完成后，你必须手动的将这些文件标记为已解决（`git add <filename>`）最后记得提交

##### 撤销合并

错了咱也有办法回退。使用`git merge --abort`可以撤销你此步的合并。而`git reset --hard`则可以回滚到合并开始前的状态

