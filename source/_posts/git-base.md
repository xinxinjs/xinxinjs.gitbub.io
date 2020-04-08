---
title: git操作
date: 2017-04-08 10:35:09
tags: git
---

### 基本操作

新建分支：

git checkout -b [分支]
git push --set-upstream origin [分支]

删除本地分支：先切换到别的分支，然后git branch -d [分支]

删除远程分支 ：git push origin --delete [分支]

切换远程仓库的地址：git remote set-url oringin <新的远程仓库URL>

查看远程仓库的地址：git remote -v

### git stash

git stash 将当前所有修改项(未提交的)暂存，压栈。此时代码回到你上一次的提交，用git status可查看状态。

git stash list将列出所有暂存项。

git stash clear 清除所有暂存项。

git stash apply 将暂存的修改重新应用，使用git status可以看到以前暂存的修改又回来了