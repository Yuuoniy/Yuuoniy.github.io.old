---
title: CTF | pwn 基础知识-栈溢出
date: 2018-08-05 09:05:12
tags:
categories: CTF
---


kernel space  
stack  
memory mapping region  
heap(libc管理)  
bss segment  
data segment  
text segment  

- register 寄存器：eax、ebx、ecx、edx、edi、esi、esp、eip、flags
- 调用约定：-64 ：rdi rsi rdx rcx r8 r9 stack 返回值：rax
- 32:stack 全部用栈传参数

- 系统调用   
特定的调用约定
  - 32：eax --syscall 号：ebx,ecx,edx 参数
  - 64:rax ---syscall 号，rdi,rsi,edx,rcx,r8,r9参数

-m32 
b test:32
32 函数调用过程:
pc rbp 
o32 没有bp
参数顺序

#### 工具
类型支持 
f5
- 流程图
- general -ida options-line prefix 开启流程图里的地址显示
- 伪代码 右键 copy to assembly 显示对应关系 光标
- shift f1

gdb 
- 装个带符号的libc
https://github.com/Escapingbug/ancypwn docker
libc -dbg

- gdb python 
- gdb cheatssheet
- GDB 可以有符号，如果有debug信息利用好
- 没有符号有源码？编译一个带debug信息 带struct信息，先编译一个带符号的.so 再把它加到这在调试的
- libc source 事半功倍
加载脚本 source
软件断点 int3 中断 signal 0xcc  程序自修改就会有问题 
和硬件断点 CPU  hb
d 断点号 
gef **ctrl+c** 在直接接受输入的地方断下来
si step instruction 可以进到got表 s不会进



**pwntools**
gdb attach
看官方文档
- IO问题很关键：read 系统调用 有可能读成两次或一次 远程timeout很玄学，因此可以通过sleep 、readline、gets,scanf 影响结果
看到read想想要不要sleep

- 遇事不决先 sleep 或 raw_input
- debug log_level 
- context
amd64 i386
context(os,arch)
cyclic pattern(cyclic(),cyclic_find())
fmtstr_payload 格式化字符串

#### 格式化字符串漏洞
print
可变参数：  
...
va_list:va_start,va_end,va_arg
从栈上往后取
污点分析
%m$X:m十进制整数，第几个参数，直接输出第几个字符，避免前面输入的干扰
- 找到控制的参数%m$X
- 利用%mX 控制输出字符数量
- %n 写入
- 任意地址写入
- 读 %s
- 用h分开 hhn 只写入一个字节
- 按内容大小读

整数溢出
注意fmtstr的构造方式，把地址放前面
- printf-chk 可以防御，%m$X %n 禁止，但无法防御读取
- sprintf,sscanf 等一切带格式化字符串的都可以采取相似思路

#### 栈溢出
保存返回地址  
ebp/rbp  
canary 有0，会进行截断。tls 操作系统生成，编译器决定
buffer  

fs 段寄存器 fs:28h 一般是用tls   
**ret2shellcode**
要求：数据可以被执行。NX,DEP
rop  
控制流变化 ret jmp call
  ROP JOP COP
xxx;ret;-gadget
ROPgadget
ret2syscall,ret2libc,ret2csu,ret2reg->不同gadget   
pop 设置寄存器的值  
ret2vdso,ret2vsyscall->特殊gadget 关注mapping  
vdso,ret2vsyscall(不能直接从syscall开始，即只能执行已经写的系统调用) 也能提供gadget 
ret2csu  --depth 20

**ret2dlresolve**
没有libc 地址。PIE:地址随机，需要调用外部函数，控制了已知地址内容。
one_gadget 地址。直接开一个shell

**ELF：**
linkable sections .bss .data,executable  segements 
权限相同 放在一起 .bss 和.data
https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/elf.h

**plt and got**
windows 导入表和导出表  

PLT:
GOT:保存已解析的地址，未解析则为下一条指令地址(jmp PLT[0])
jmp GOT; push 。。

目的：伪造字符串，伪造index plt 中 push。
reloc:基地址+Index .控制reloc结构体

已知buffer可以控制，放结构体 
伪造的reloc 来伪造sym.将sym放在已知的位置。strtab +st_name 得到能控制字符串的地址。

partial overwrite syscall 

**SROP**
sigreturn 
控制PC-继续sigreturn -rop
只需要控制ax和sp
要用syscall

pwntools sigreturn 

先调用write泄露栈的地址
栈顶
调用syscall exec bin/sh 在已知的地址


怎么泄露栈顶的地址？？？

stack pivot:rbp ->rsp 
tls
canary:fs 0x28 (fs的值拿不到)
arch_prctl brop

