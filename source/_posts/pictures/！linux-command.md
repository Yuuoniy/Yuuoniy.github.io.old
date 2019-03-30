---
title: linux基础命令学习
date: 2017-08-16 14:22:05
tags: linux
categories: 学习
---
有点乱的
<!-- more -->

date +(显示格式)
cal (月份)(年份)
bc 简单计算器 
ctrl+d 通常表示键盘输入结束 
sync 数据同步写入磁盘 
rmdir 删除空的目录 
basename/dirname 取得路径的文件名和目录名  
tac 从最后一行开始显示 
more 一页一页显示档案的内容 
less 在more的基础上 可以往前翻页 
umask 档案预设权限 
file 查看文件类型 


挂载：
分配盘符 

/etc/fstab 自动挂载设置 
#! /bin/bash 
光盘文件系统iso9660  
umount 卸载 
格式化： 写入文件系统 数据块 

/etc/shells 查看可用的shell 
echo $SHELL 查看当前的shell  

echo -e 特殊符号 
echo 中 ！有特殊含义
echo 中 "\e[1;颜色对应符号 输出内容 \e[0m"  可以控制输出颜色  
30m 黑色
31m 红色
32m 绿色...

 
脚本执行：
1. chmod 755 脚本名称;(赋予执行权限)  ./脚本
2. bash 脚本 

alias 查看别名 
alias 别名=‘命令’ 临时生效 
写入 vi ~/.bashrc 写入环境变量配置文件 (对应root用户)
source .bashrc 当即生效 
unalias 接触  

快捷键：
ctrl + l 清屏 
ctrl + a 移到行首 
ctrl + e 移到行尾 
ctrl + u 光标所在位置删除到行首 
ctrl + r 在历史命令中搜索  
ctrl + z 把命令放在后台  没有终止..


#### 历史命令 
history 
保存在 ~/.bahs_history 
history -w 把当前的历史命令写入文件(否则等用户退出才写入)  
-c 清空历史命令 

！n 执行第n条历史命令 
！！重复执行上一条历史命令 
!字符串 

#### 输出重定向  
\> 以覆盖方式 
\>>以追加方式 
ifconfig 网卡信息 
2>文件 (左右两侧没有空格) 标准错误信息 

2>&1
&>/dev/null 把输出内容都丢了  
netstat -an 查看服务器连接了多少客户端 

