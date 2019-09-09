---
title: 初识Git
categories: Git
tags: 
 - 版本控制
 - 版本库
---

`Git`是每个程序员都应该会使用的、必不可少的工具，所以在学习的过程中我会将笔记推送至此便于日后的学习和温故。

<!--more-->

### 版本控制

#### 集中式版本控制系统

**将鸡蛋放在同一个篮子里**的版本控制系统。

- 必须联网才能工作
- 必须需要一台中央服务器

由此就带来了诸多缺陷，比如在互联网环境下不可保证的网速，及对服务器性能的高要求，以及对后端的强依赖。如此一来分布式应运而生。

#### 分布式版本控制系统

分布式则可以理解为各自为战，每个人的电脑上都是一个完整的版本库。多方协作时只需将各自的修改推送给对方就可以看到对方的修改了。

与集中式相比分布式版本控制系统安全性要高得多，不必担心服务器崩溃导致的数据丢失。毕竟不是依赖中央服务器。本机数据丢失后再从其他人那里复制一份文件就可以了。

### 安装

#### Linux

```
sudo apt-get install git
```

#### Mac OS

```
在 AppStore 安装 Xcode 。
需要使用时运行 Xcode ，选择菜单"Xcode" -> "Preferences"，在弹出窗口中找到"Downloads"，选择"Command Line Tools"，点击"Install"
```

#### Windows

去官网上下载就好啦。

安装完毕后在命令行输入

```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

### 版本库

> 我们可以通过`Git`管理版本库中的所有文件，包括其中每一个文件的修改、删除、追踪和远远

#### 创建

```
$ mkdir learngit
$ cd learngit
$ pwd
/Users/michael/learngit
```

`pwd`用于显示当前目录

通过`git init`在此目录添加一个`.git`目录。这个`.git`目录是`Git`用来跟踪管理版本库。

若你没有看到`.git`目录则说明系统将其隐藏了。通过`ls -ah`就可以看到啦

#### Windows 不要用记事本编辑文件

要强调的都在题目上了，这点切记

微软自作聪明的在每个文件开头添加了`0xefbbbf`的字符，所以你可能会碰到很多不可思议的事情

#### 添加文件

我们可以新建一个`readme.txt`文件，内容写这个

```
Git is a version control system
Git is free software
```

我们可以将所需管理的文件添加到我们刚刚所创建的文件夹`learngit`下，再**利用`git add`命令告诉`Git`，将文件添加到仓库**

```
$ git add readme.txt
```

再**用`git commit`告诉`Git`，把文件提交到仓库**

```
$ git commit -m "wrote a readme file"
[master (root-commit) eaadf4e] wrote a readme file
 1 file changed, 2 insertions(+)			// 1个文件被改动（aeadme.txt），插入了2行内容
 create mode 100644 readme.txt
```

> **`-m "xxx"`是对此次提交的说明**，务必要写！以后方便自己查询。

`add`一次只能添加一个文件，而`commit`则可以一次提交多个文件

```
$ git add file1.txt
$ git add file2.txt file3.txt
$ git commit -m "add 3 files."
```