---
title: CTF | jarvisoj-writeup
date: 2018-08-17 20:55:06
tags: pwn
categories: CTF
---
jarvisoj 上的题目
<!-- more -->
## levle2-x64
差别在于32位和64位传参

64位参数传递约定：前六个参数按顺序存储在寄存器` rdi, rsi, rdx, rcx, r8, r9`中
参数超过六个时，从第七个开始压入栈中

所以只需要在调用函数传递参数 `"bin/sh"` 时，将其传入寄存器即可
使用 `rop` ,`pop rdi ret;` 即为将栈顶元素弹出并存入寄存器rdi，ret返回栈。

## level3_x64
需要自己设置好寄存器 按顺序为` rdi,rsi,rdx `
构造 `syscall` 
```
rdi - bin/sh
rsi - 0
rdx - 0
```
