---
title: windows10连接远程桌面ubuntu16.04
date: 2017-07-28 13:21:27
tags: linux
categories: 学习
---
<!--more-->
在ubuntu上：
`sudo apt-get install xrdp vnc4server xbase-clients`  

通过搜索打开desktop sharing  
按照下图设置: 

![](http://otswdapxf.bkt.clouddn.com/desktopSharing.jpg?imageView/2/w/500) 

安装dconf-editor:  

`sudo apt-get install dconf-editor`

打开：  
`dconf-editor`
设置：org > gnome > desktop > remote-access，取消 “requlre-encryption”  


![](http://otswdapxf.bkt.clouddn.com/%E8%AE%BE%E7%BD%AE.png?imageView/2/w/500)

在windows中 打开远程桌面连接,输入ip地址   

![](http://otswdapxf.bkt.clouddn.com/%E8%BF%9E%E6%8E%A5.png?imageView/2/w/400)

切换到如下界面： 
选择vnc-any，输入ip地址，端口不变,输入刚刚在ubuntu上设置的密码:  


![](http://otswdapxf.bkt.clouddn.com/%E8%BF%9E%E6%8E%A52.png?imageView/2/w/500) 

连接成功 ！好开心 ！  

![](http://otswdapxf.bkt.clouddn.com/%E8%BF%9E%E6%8E%A5%E6%88%90%E5%8A%9F.png?imageView/2/w/500)

然后发现 两个鼠标不同步，十分不好用，再控制面板的鼠标设置中取消了 提高指针精确度  
也没好多少..  
又自己下载了VNC 好了一些,不过还是卡卡的  
(具体的VNC设置就不说啦，就一步一步根据提示来吧，不难)  
