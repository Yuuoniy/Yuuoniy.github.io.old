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
git reset --hard HEAD (撤销合并)


git diff --staged   

git pull upstream/master     
git add -u
注：-u 表示把所有已track的文件的新的修改加入缓存，但不加入新的文件。
git show （与父提交比较  
git branch -d 删除某个分支（只是删除这个标签）  
git merge 始终将所有指定的分支合并到当前检出的分支中，并为该分支新建一个提交。  
git merge --abort，将文件恢复到你开始合并之前的状态  

#### log ####
git log --graph   看到分支合并图
git log --graph --pretty=oneline --abbrev-commit 可以展示合并信息
git log --stat   显示在每个提交(commit)中哪些文件被修改了,这些文件分别添加或删除了多少行内容.  
git log --pretty=oneline (pretty参数格式化)
git log --pretty=short
--no-ff参数，表示禁用Fast forward  
--allow-unrelated-histories
记录的命令操作很不系统，但主要还是想记录自己不熟悉的内容（慢慢也会变熟悉的）
gitk  查看图形化信息

解决git在windows下中文乱码:
```
$ git config --global core.quotepath false          # 显示 status 编码
$ git config --global gui.encoding utf-8            # 图形界面编码
$ git config --global i18n.commit.encoding utf-8    # 提交信息编码
$ git config --global i18n.logoutputencoding utf-8  # 输出 log 编码
$ export LESSCHARSET=utf-8
# 最后一条命令是因为 git log 默认使用 less 分页，所以需要 bash 对 less 命令进行 utf-8 编码

```


