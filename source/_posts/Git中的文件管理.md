---
title: Git中的文件管理
categories: Git
tags:
 - 版本回退
 - 管理修改
 - 撤销修改
 - 删除文件
---

今天我们试着对已经提交了的文件进行操作。

<!--more-->

### 修改

- 修改文件
- `$ git status`：查看当前状态
- `$ git diff `：查看修改的具体内容
- `$ git add xxx.xx`：将修改后的`xxx.xx`文件添加到仓库
- `$ git status`：查看当前状态
- `$ git commit -m "Description Message"：提交到仓库

### 版本回退

- `$ git log`：查看近期提交日志
- `$ git log --pretty=oneline`：查看近期提交日志（减少输出信息）
- `$ git reset --hard HEAD^ `：回退到上一个版本
- `$ git reset --hard HEAD^^`：回退到上两个版本
- `$ git reset --hard HEAD~100`：回退到上一百个版本
- `$ git reset --hard 1094a`：回退到`1094a`版本（此处的版本号是`commit id`，我们如果回退的版本过前想再回来点就只能输入制定的版本号来进行回退了。版本号没必要写全，但也得保证至少5位）

### 工作区和暂存区

- `.git`隐藏目录就是`Git`的版本库。里边存了很多东西，其中就包括暂存区（`stage/index`）、分支（`master`）及指向`master`的指针`HEAD`。

- 每当我们使用`$ git add xxx.xx`命令时会将`xxx.xx`文件推至暂存区（`stage/index`）

![](https://pic.superbed.cn/item/5ca9c27a3a213b0417d24eb9)

- 而当我们使用`$ git commit xxxx.xx`命令时会将`xxx.xx`文件从暂存区（`stage`）推至分支（`master`）。此时暂存区的内容就被清空啦

![](https://pic.superbed.cn/item/5ca9c25a3a213b0417d24d5f)

### 撤销修改

#### 未添加到暂存区

- `$ git checkout -- xxx.xx`：丢弃工作区的修改

#### 已添加到暂存区

- `$ git reset HEAD xxx.xx`：将暂存区的修改撤销掉（`unstage`），重新放回工作区
- 再通过`$ git checkout -- xxx.xx`：丢弃工作区的修改

### 删除文件

- 删除文件后使用`$ git status`命令会立即告诉你哪些文件被删除了
- `git rm`后，确定从版本库中删除该文件，就`$ git commit -m "remove xxx.xx"`
- `$ git checkout -- xxx.xx`：误删恢复

### 远程仓库

往往通过选取一台电脑充当**服务器**的角色，其他人都从这个服务器仓库克隆一份到自己电脑上，并且把各自的提交推送到服务器仓库里。也从服务器仓库中拉取别人的提交。如此就起到了类似于**集中式版本控制系统**的作用。

我们可以用`Github`当作这个服务器。由于`Git`和`GitHub`之间的传输是通过 SSH 加密的，所以需要一点设置：

#### 创建 SSH Key

在用户主目录下查看是否有`.ssh`目录。

- 若有则再看看此目录下是否有`id_rsa`和`id_rsa.pub`两个文件。
  - 有则跳到下一步。
  - 无则在`Windows`下打开`Git Bash`，创建`SSH Key`。 —— `$ ssh-keygen -t rsa -C "youremail@example.com"`

之后就是一路回车，也无需设置`Key`。

#### 连接GitHub

登陆`GitHbu`，点击头像，打开`Settings`，`SSH and GPG keys`页面，点击`New SSH key`。

将用户主目录（`Windows`下是`C:\Users\yourUserName\.ssh`）下的`id_rsa.pub`的密钥复制上去，名字你自己随便取。

#### 添加远程库

- 点击**头像旁边的加号**，选`New repository`，新建新存储库。
- 存储库名称和自己要上传的文件名保持一致（**比如叫`learngit`**），其余的不用管，采取默认设置即可。
- 如此就完成了添加新仓库的操作，但是目前为止这个新仓库还是空的

我们可以从这个仓库克隆出新的仓库，也可以把一个已有的本地仓库与之关联，然后将本地仓库的内容推送到`Github`仓库。

- 我们可以根据提示，在本地的`learngit`仓库下运行命令

- ```
  									    your GitName  fileName
  $ git remote add origin git@github.com:Burning-Shadow/learngit.git
  ```

- 添加后远程库的名字默认就是`origin`。

- ```
  // 将本地库的所有内容推送到远程库（将 master 推送到远程新的 master 分支，并将本地和远程的 master 分支关联起来）
  // 第一次推送 master 分支时添加 -u 参数
  $ git push -u origin master
  ```

- 如此以来即可在`GitHub`上查看

#### 关联后

- 至此，只要本地做了提交就可以通过`$ git push origin master`将本地`master`分支的最新修改推送至`GitHub`。

### 从远程库克隆

远程仓库部分讲解了现有本地库后有远程库的时候如何关联程序

但若我们从零开发，那么最好的方式是创建远程仓库，然后从远程库克隆

- 首先登陆`GitHub`，创建一个新的仓库（假设名为`gitskills`）

- 在新建前记得勾选`Initialize this repository with a README`。如此`GitHub`会自动为我们创建一个`README.md`文件

- 用命令克隆一个本地库

- ```
  $ git clone git@github.com:Burning-Shadow/gitskills.git
  ```

- 然后进入`gitskills`目录看，已经有`README.md`了

- 多人协作开发，那么每个人各自从远程克隆一份就可以了