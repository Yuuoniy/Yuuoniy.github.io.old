---
title: CTF |  kernel pwn 的资料及教程
date: 2018-09-17 10:55:36
tags:
---
源码阅读：
http://www.makelinux.net/kernel_map/
https://elixir.bootlin.com/linux/latest/source/kernel


资料补充：
[深入理解linux系统下proc文件系统内容](http://www.cnblogs.com/cute/archive/2011/04/20/2022280.html)


搭建源码环境：
gdb+quem
https://github.com/google/syzkaller/blob/master/docs/linux/setup_ubuntu-host_qemu-vm_x86-64-kernel.md


### 驱动
首先学习一下Linux 驱动相关
需要理解 内核空间和用户空间。应用程序驻留在用户空间, 模块和设备驱动驻留在内核空间

通常，用户空间的每个函数（用于使用设备或者文件的），在内
核空间中都有一个对应的功能相似并且可将内核的信息向用户传
递的函数。


相关资料：
http://read.pudn.com/downloads119/ebook/506573/Linuxdevicedriver.pdf

#### 在用户空间加载和卸载驱动
命令：
`insmod + filename` 安装模块
`lsmod` 查看已安装的模块
`rmmod` 溢出模块

内核空间的函数
`module_init` 对应insmod
`module_exit` 对应rmmod

printk 函数，在内核中有效，其中<n> 表示优先级
查看系统日志文件 `cat /var/log/syslog`

#### 驱动模块表
```
Events	User functions	Kernel functions
Load	insmod	module_init()
Open	fopen	file_operations: open
Close	fread	file_operations: read
Write	fwrite	file_operations: write
Close	fclose	file_operations: release
Remove	rmmod	module_exit()
```

#### 

题目：
[记录强网杯2018一道内核pwn的解题思路](https://www.anquanke.com/post/id/103920)

[linux-kernel expoit study（1） ---编译并用qemu运行内核]https://bestwing.me/2017/04/04/Complie-linux-kernel-and-running-it-using-qemu/

#### 环境搭建
