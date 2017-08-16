---
title: ubuntu16.04安装chrome
date: 2017-07-29 13:20:44
tags: linux
categories: 学习
---

<!-- more -->  
将下载源加入系统的源列表：  
`sudo wget http://www.linuxidc.com/files/repo/google-chrome.list -P /etc/apt/sources.list.d/`  
如果地址失效可以搜索其他下载源  

![](http://otswdapxf.bkt.clouddn.com/chrome1.jpg?imageView/2/w/500)

导入谷歌的公匙:  

`wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -`

![](http://otswdapxf.bkt.clouddn.com/chrome2.jpg?imageView/2/w/500)


更新列表：  
`sudo apt-get update`

安装：  
`sudo apt-get install google-chrome-stable`
