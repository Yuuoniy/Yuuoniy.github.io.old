---
title: gonme安装GTK主题ARC(ubuntu16.04)
date: 2017-07-29 12:01:45
tags: linux
categories: 学习
---

<!-- more -->
[Arc](https://github.com/horst3180/arc-theme)  

安装gnome-tweak-tool:  
`sudo apt install gnome-tweak-tool`

安装GTK3：  
`sudo apt-get install build-essential`(如果还没有)  
`sudo apt-get install libgtk-3-dev`   

获取主题并编译安装：   
`git clone https://github.com/horst3180/arc-theme --depth 1 && cd arc-theme`  
`sudo apt-get install dh-autoreconf`  
`sudo make install`

打开gnome-tweak-tool：  
`gnome-tweak-tool`  

设置主题样式为ARC:  
![](http://otswdapxf.bkt.clouddn.com/set-theme.jpg?imageView/2/w/500)

但是...没有实现透明...

具体设置可看主题链接  
  
