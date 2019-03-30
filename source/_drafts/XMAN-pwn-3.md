---
title: CTF | IO_FILE 学习笔记
date: 2018-08-07 08:55:02
tags:
---
##__IO__FILE
###history of __
- seethefile
- house of orange
- WCTF 2017 wannaheap libc-2.24.so a NULL byte overwrite
- parrot tokoy wester
- WHCTF stackoverflow
- echo_back ciscn 
###  what is
- libc-2.26.so
- 阅读源码
- `_IO_list_all` 链表的头 可读可写的区域 _IO_list_all-stderr-stdout-stdin
- _io_file_plus 多了vtable 是个虚表，包含函数地址
- vtable is a _IO_jump

文件操作
#### fread 
- `size_t fread ( void *buffer, size_t size, size_t count, FILE *stream)` ;
- `_IO_XSGETN` 虚表中的一个函数
- 
#### fwrite
#### printf/puts
  都会用到`_IO_file_xsputn`
  gcc 

  fp 设置成buf的地址 rbx

  首先看到crash在cmp r2,qword ptr[rdx+8]的地方。

  rdi+0x88 :`fp->_lock` 如果是0就不进行这段比较
  **seethefile**
  


   的buf，
  再次 crash，看 crash 地址，让 loca->pointer=0
  `vtable` 设置在伪造fp的后面，因为文件操作会有一些检查
，try not use the fp pointer internal data 
aslr:要泄露libc地址，
要注意`_vtable_offset`要为`0`，其偏移为`0x46`且只占一个字节  
### house of orange
请看 house of range 单独的博文

##### house of lemon
stdout 在 main arena,变成释放的advice的堆块

通过stdout vtable 劫持控制流，但是没法设置参数
技巧：IO_str_jumps 另外一个虚表 

stdin、stdout正好被编译在main_arena后面，就是说如果fastbin的尺寸合适是可以溢出main_arena覆盖到后面stdout\stdin结构的。我们知道堆中释放的内存只是逻辑上的释放，实际上的内存映射并不会取消，那么我们可以先在合适尺寸的堆中填入伪造的函数指针，再释放这个堆就可以溢出main_arena覆盖标准流的vtable从而劫持程序流程。

https://zhuanlan.kanxue.com/article-446.htm
p *stdout
io_str_jumps
telescope 
new size 变成bin/sh 的地址
easy python decompiler 
**parrot**
泄露libc 地址，
init free 基址：free一个堆块，如果前面有fastbin，如果再add small bin ,检查前面有没有 fast bin，合并，放到unsorted bin，
heap 
参考链接：
https://github.com/scwuaptx/CTF/blob/master/2017-writeup/twctf/parrot.py
http://shimasyaro.hatenablog.com/entry/2017/09/19/103327
https://github.com/scwuaptx/CTF/blob/master/2017-writeup/twctf/Parrot.md
http://brieflyx.me/2017/ctf-writeups/twctf3-2017-parrot/
null byte 任意写

https://bbs.pediy.com/thread-223334.htm  从BookWriter看house_of_orange原理【新手向】	
http://p4nda.top/2018/05/13/ciscn-ctf-2018/ echo back
http://4ngelboy.blogspot.com/2016/10/hitcon-ctf-qual-2016-house-of-orange.html   
https://www.slideshare.net/AngelBoy1/advanced-heap-exploitaion
https://www.xctf.org.cn/library/details/54d5bc6b64467a4fd8f21cb2d5cb53a4fe5ae477/

house of range:
http://tacxingxing.com/2018/01/10/house-of-orange/#toc_6



### 伪造vtable劫持程序流程
_IO_FILE_plus结构中存在vtable，一些函数会取出vtable中的指针进行调用。
中心思想就是针对_IO_FILE_plus的vtable动手脚，通过把vtable指向我们控制的内存，并在其中布置函数指针来实现
vtable劫持分为两种，一种是直接改写vtable中的函数指针，通过任意地址写就可以实现。另一种是覆盖vtable的指针指向我们控制的内存，然后在其中布置函数指针
### FSOP
File Stream Oriented Programming
进程内所有的`_IO_FILE`结构会使用_chain域相互连接形成一个链表，这个链表的头部由_IO_list_all维护
劫持_IO_list_all的值来伪造链表和其中的_IO_FILE项
触发方法是调用`_IO_flush_all_lockp`
这个函数会刷新`_IO_list_all`链表中所有项的文件流，相当于对每个FILE调用fflush，也对应着会调用_IO_FILE_plus.vtable中的`_IO_overflow`
在一些情况下这个函数会被系统调用：
当libc执行abort流程时
当执行exit函数时
当执行流从main函数返回时
### 新版本libc下IO_FILE的利用
`glibc_2.24`以后，全新加入了针对IO_FILE_plus的vtable劫持的检测措施，`glibc` 会在调用虚函数之前首先检查vtable地址的合法性
关注点从`vtable`转移到`_IO_FILE`结构内部的域中
进程中包含了系统默认的三个文件流`stdin\stdout\stderr`，因此这种方式可以不需要进程中存在文件操作，通过`scanf\printf`一样可以进行利用
在_IO_FILE中`_IO_buf_base`表示操作的起始地址，`_IO_buf_end`表示结束地址，通过控制这两个数据可以实现控制读写的操作

vtable 的地址怎么算出来？就是算出来伪造的`vtable`在堆中的地址，通过动态调试可以看到的
TokyoWesterns 2017 - Parrot

### FSOP
劫持` _IO_list_all` 来伪造链表的利用技术，通过调用 `_IO_flush_all_lockp()`触发，该函数函数下面三种情况下被调用：
- libc 检测到内存错误时
- 执行exit函数时
- mian 函数返回时

两条FSOP路径
1.` malloc err`
  `_IO_OVERFLOW(fp, EOF)` 的执行过程中最终会调用 `system('/bin/sh')。`
2. 关闭流
  `_IO_FINISH (fp)` 的执行过程中最终会调用 `system('/bin/sh')。`
  此时需要设置`_lock` 指向内容为`0`的地方，`64`位中偏移是`0x88`
#### libc-2.24防御机制
1. IO_validate_vtable 
2. _IO_vtable_check 
利用技术
- _IO_str_jumps
这个 vtable 中包含了一个叫做 _IO_str_overflow 的函数，该函数中存在相对地址的引用（可伪造）

**bookwriter**

ebp的内存单元存储的是saved ebp，上一个函数的ebp值，该值是栈的地址
Pie就是随便一个变量的pie地址，减去相对偏移，得到程序基址

seethefile:
https://www.jianshu.com/p/2e00afb01606
透过bookwriter学习top chunk和_IO_FILE利用 https://blog.csdn.net/weixin_40850881/article/details/80043934
