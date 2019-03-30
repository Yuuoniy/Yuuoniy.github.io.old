---
title: CTF | 2018 护网杯 shoppingCart
date: 2018-10-17 23:45:17
tags:
---
没写完呢。。
<!-- more -->
太菜了。。这道题比赛时都做不出来

## 程序分析
有数组越界读写，在对 goods 进行 modify 时可以访问数组以外的数据，8字节，通过越界可以访问到堆地址信息
bss 段 goods数组存储 malloc 得到的 good结构的指针信息 
good 结构保存：name  ptr(malloc) ，num 信息
程序开了 PIE, Partial RELRO 可以修改 got 表


## 漏洞利用
modify 越界，而且可以负数越界!
我一直在纠结 读取字符串会加\x00
printf 有 \x00 截断怎么 leak 地址。。之后才知道 size = 0就好了。。
### 泄露 libc 
首先创一个 unsorted bin，写入 fd,bk ，可以 add 一个 size 为 0 的，此时会从 unsorted bin 中割出来一块 0x20 的chunk,写入 fd,bk 。因为 size 为 0,所以也不会覆盖零字节，就可以泄露地址了...！(好菜啊 

### 劫持控制流
我们通过 modify 函数，

## 脚本


