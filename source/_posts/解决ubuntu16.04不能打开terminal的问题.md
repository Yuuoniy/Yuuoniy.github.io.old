---
title: 解决ubuntu16.04不能打开terminal的问题
date: 2017-07-28 14:44:49
tags: linux
categories: 学习 
---
挺简单的
<!--more-->
语言设置的问题  
执行：  
`sudo apt-get update ` (可以直接用apt代替 apt-get吧)  
`sudo apt-get upgrade` (这其中出现一大丢看不懂的更新 还让我设置的..硬着头皮..过去了)  

(上面两步并不是非常必须的吧)  
`locale-gen` (生成locale)  
`localectl set-locale LANG="en_US.UTF-8`  

然后！重启！  
`sudo reboot`  
