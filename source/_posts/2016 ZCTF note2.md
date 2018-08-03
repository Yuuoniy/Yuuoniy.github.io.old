---
title: 2016 ZCTF note2
date: 2018-04-05 23:27:02
tags:
---

首先，我们先分析一下程序，可以看出程序的主要功能为

添加note，size限制为0x80，size会被记录，note指针会被记录。
展示note内容。
编辑note内容，其中包括覆盖已有的note，在已有的note后面添加内容。
释放note。
仔细分析后，可以发现程序有以下几个问题

在添加note时，程序会记录note对应的大小，该大小会用于控制读取note的内容，但是读取的循环变量i是无符号变量，所以比较时都会转换为无符号变量，那么当我们输入size为0时，glibc根据其规定，会分配0x20个字节，但是程序读取的内容却并不受到限制，故而会产生堆溢出。
程序在每次编辑note时，都会申请0xa0大小的内存，但是在 free 之后并没有设置为NULL。

https://ctf-wiki.github.io/ctf-wiki/pwn/heap/unlink/


