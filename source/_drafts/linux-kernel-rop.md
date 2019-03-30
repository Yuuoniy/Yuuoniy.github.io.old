---
title: CTF | 内核 ROP 
date: 2018-09-17 15:59:46
tags:
---
递归学习。。。学着学着，发现我这也不懂，那也不懂...
<!-- more -->
相关博客：
http://pwn4.fun/2017/06/28/Linux-x64%E5%86%85%E6%A0%B8ROP/ 内核ROP
https://www.w0lfzhang.com/2017/08/06/Linux-Kernel-ROP/ 内核ROP
https://www.trustwave.com/Resources/SpiderLabs-Blog/Linux-Kernel-ROP---Ropping-your-way-to---(Part-1)/
http://pwn4.fun/2017/06/28/Linux-x64%E5%86%85%E6%A0%B8ROP/  Linux x64内核ROP

基本思路：
通过ROP来改变CR4的值来关闭SMEP，然后就可以在用户空间执行payload提权了。

rop:
```
|----------------------|
| pop rdi; ret         |<== low mem
|----------------------|
| NULL                 |
|----------------------|
| addr of              |
| prepare_kernel_cred()|
|----------------------|
| mov rdi, rax; ret    |
|----------------------|
| addr of              |
| commit_creds()       |<== high mem
|----------------------|
```
smep_bypass:
https://cyseclabs.com/slides/smep_bypass.pdf 
ioctl 函数
tty_struct结构体 

http://p4nda.top/2018/07/13/ciscn2018-core/
https://whereisk0shl.top/NCSTISC%20Linux%20Kernel%20pwn450%20writeup.html
所谓smep是内核为了避免ret2user的利用方法增加的一种保护方法，即内核代码不能跳转到用户空间去执行代码，绕过方法也很简单，使用内核的ROP就可以了


由于kernel pwn 的最终目的是提权到root，一种简单的方法是执行

1
commit_creds(prepare_kernel_cred(0));
而commit_creds、prapare_kernel_cred都是内核函数，在vmlinux中，因此还需要泄露vmlinux的基地址。

