---
title: ubuntu16.04安装桌面系统gnome
date: 2017-07-28 11:17:52
tags: linux
categories: 学习
---
执行命令：  
`sudo apt-get update`  
`sudo apt-get gnome-desktop`  

执行`startx`启动图形化界面(好像通过的方式这样会导致下面的问题...)   

我遇到了登录后只显示壁纸的情况，解决方法：  
打开一个终端(例：ctrl+alt+f2)  
执行命令  
`rm -rf ~/.compiz ~/.cache`  
`sudo systemctl restart lightdm`  
即可恢复正常
(其中的原因我并不了解)  

添加用户到sudo用户组：  
登录一个sudo账户：`sudo usermod -G sudo 用户名`  
通过groups 用户名可以查所属的用户组

还有一些因人而异的问题...
