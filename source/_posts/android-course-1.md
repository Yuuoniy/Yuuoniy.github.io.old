---
title: android | 手机平台应用开发第一篇
date: 2018-09-11 14:47:50
tags: android
categories: 
---
<!-- more -->
### git 日常使用总结
git 已经用很多了,大概就是需要了解仓库,分支等概念,熟悉本地和远程的关系,熟练使用clone commit pull push merge操作
下面给出一下常用的命令:
新建分支:`git branch dev`
切换分支:`git checkout dev`
添加文件 `git add file` ,常用 `git add .` 添加全部更新的文件
查看状态 `git status`
取回远程仓库的变化，并与本地分支合并:`git pull [remote] [branch]`
上传本地指定分支到远程仓库:`git push [remote] [branch]`
常见步骤:`git add ->commit ->push`, 如果有冲突还涉及到 merge 操作
显示版本历史:`git log`
显示暂存区和工作区的不同: `git diff`
对于入门者推荐 git 教程 [如何使用 Git 和 GitHub](https://classroom.udacity.com/courses/ud775)
比较系统,之前我也是根据这个学的

### Android开发环境搭建
之前已经安装过 AS 了,下载sdk应该要翻墙,不过我有梯子,不过 sync gardle 时总是失败
错误：
`Unknown host 'dl.google.com'. You may need to adjust the proxy settings in Gradle.`
按照网上的方法:
`File > Settings > Appearance & Behavior > System Settings > HTTP Proxy [Under IDE Settings] Enable following option Auto-detect proxy settings` 
没有解决
很奇怪,在 setting里面尝试连接谷歌是成功的
尝试 `enable embedded maven repo` 依旧失败

又出现 `failed to find build tools revision x.x.x` 的问题
install build tool 后
依旧是  `gardle project sync failed` 的问题
结果发现翻了墙还是要改 host 呀，修改 Host 就好了，连接手机，完美运行app
