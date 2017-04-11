---
title: git入门学习
date: 2017-04-10 07:16:05
tags: git
categories: 学习
---
git相关配置及基础命令
<!-- more -->

配置简写：

```
$ git config --global alias.co checkout
$ git config --global alias.ci commit
$ git config --global alias.br branch

```
git config --global core.autocrlf true。 修复因换行符不同产生的问题（对于windows)  
git reset 从暂存区删除  
git reset --hard 放弃工作目录或暂存区的所有更改（不能恢复的！！  
git diff --staged   

git pull upstream/master     
git show （与父提交比较  
git branch -d 删除某个分支（只是删除这个标签）  
git merge 始终将所有指定的分支合并到当前检出的分支中，并为该分支新建一个提交。  
git merge --abort，将文件恢复到你开始合并之前的状态  
git log --graph   
git log --graph --pretty=oneline --abbrev-commit 可以展示合并信息
--no-ff参数，表示禁用Fast forward  

记录的命令操作很不系统，但主要还是想记录自己不熟悉的内容（慢慢也会变熟悉的）
