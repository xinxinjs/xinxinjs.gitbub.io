---
title: git-rebase
date: 2020-04-08 10:45:34
tags: [git]
---

太多的commit不利于code review，造成分支污染，如果需要回滚代码，发现有很多的commit，难以定位要回滚的版本

## 使用场景：合并多次commit

git rebase -i HEAD~4

进入vi模式：

- p, pick = use commit

- r, reword = use commit, but edit the commit message

- e, edit = use commit, but stop for amending（停下来修改）

- s, squash = use commit, but meld into previous commit（融入先前哪个commit）

- f, fixup = like “squash”, but discard this commit’s log message（丢弃这个commit的日志信息）

- x, exec = run command (the rest of the line) using shell（运行命令）

- d, drop = remove commit

## git pull --rebase

多个人使用同一个远程分支合作开发，执行git pull之后，远程有新的commit

## git rebase --abort
