---
title: git学习整理
date: 2022-09-25 20:06:06
categories: 
- svn
tags: 
- git
---

# 1 创建版本库

版本库又名仓库，英文名repository，你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

初始化一个Git仓库，使用`git init`命令。

添加文件到Git仓库，分两步：

1. 使用命令`git add 文件名`，注意，可反复多次使用，添加多个文件，此时文件放到了缓存区；
2. 使用命令`git commit -m "提交信息"`，完成，此时文件从缓存区被放到了head指定的当前分支中。

# 2 查看工作区状态与提交修改

要随时掌握工作区的状态，使用`git status`命令。

如果git status告诉你有文件被修改过，用`git diff`可以查看修改内容。

提交修改与添加文件的步骤一样

```
 git add 文件名
 git commit -m "add a line"
```

将暂存区的版本提交到版本库,从而形成工作区->暂存区->版本库的基本链路,本地工作区的版本控制流程大致如此.

# 3 版本切换

HEAD指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`（commit_id可以输入前几位就行）。

穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。

要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。

# 4 多次修改

第一次修改 -> git add -> 第二次修改 -> git add -> git commit

# 5 修改撤销

- 场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。
- 场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD` ，就回到了场景1，第二步按场景1操作。
- 场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考**版本切换**一节，不过前提是没有推送到远程库。

# 6 删除文件

`rm test.txt`  

如果在本地删除了文件，可以通过以下操作在git库里删除  

`git rm test.txt`  

`git commit -m "remove test.txt"`  

可以通过以下操作恢复  

`git checkout -- test.txt`

# 7 添加远程库

把本地库的内容推送到远程，用git push命令，实际上是把当前分支master推送到远程。  

由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

1. 要关联一个远程库，使用命令git remote add origin git@server-name:path/repo-name.git；
2. 关联后，使用命令git push -u origin master第一次推送master分支的所有内容；
3. 此后，每次本地提交后，只要有必要，就可以使用命令git push origin master推送最新修改；
