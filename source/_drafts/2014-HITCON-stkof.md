---
title: 2014-HITCON-stkof
date: 2018-04-04 23:27:02
tags:
---
乍一看不知道程序是干嘛的，那我再好好看一下。
好了放过自己，先看一下别人的吧，毕竟还在学。
不知道为什么有 :: 这反汇编好奇怪啊！
atol()函数：将字符串转换成long
atoll函数：将字符串转化为long long类型变量


程序几乎啥输出也没有，只能硬看了，大概是一个内存分配器，差不多有四个功能

1，分配指定大小的内存，并在bss段记录对应 chunk 的指针，假设其为global。
2，根据指定索引，以及指定大小向指定内存处，读入数据。可见，这里存在堆溢出的情况，因为这里读入字节的大小是由我们来控制的。
 `fgets(&s, 16, stdin); n = atoll(&s);`
3，根据指定索引，释放已经分配的内存块。
4，这个功能并没有什么用

值得注意的是，由于程序本身没有进行 setbuf 操作，所以在执行输入输出操作的时候会申请缓冲区。 什么意思？

有很明显的堆溢出，输入的东西可以大于分配的空间。

由于程序本身没有 leak，要想执行 system 等函数，我们的首要目的还是先构造 leak，基本思路如下

利用 unlink 修改 global[2] 为 &global[2]-0x18。 为什么是改为-0x18=24 我知道了
利用编辑功能修改 global[0] 为 free@got 地址，同时修改 global[1] 为puts@got 地址，global[2] 为 atoi@got 地址。
修改 free@got 为 puts@plt 的地址，从而当再次调用 free 函数时，即可直接调用 puts 函数。这样就可以泄漏函数内容。
free global[2]，即泄漏 puts@got 内容，从而知道 system 函数地址以及 libc 中 /bin/sh 地址。
修改 atoi@got 为 system 函数地址，再次调用时，输入 /bin/sh 地址即可。


通过unlink漏洞,修改free@got为puts,输入puts@plt,输出puts函数的真实地址.修改atoi@got为system, 输入/bin/sh的地址, 获得shell. 
使用pwntools，利用unlink漏洞，改写free为puts，实现任意地址泄露。然后使用DynELF找到system，再将free替换为system，执行system(‘/bin/sh’)。

还是有一些东西不懂

Allocate 2 buffers.
Overflow the first buffer into the second buffer.
Setup the data structures so that free() doesn't error out and calls unlink() with our wanted values.
Free the 2nd buffer.
Call readin(idxb, ptrto) to put the destination pointer (a GOT address) in the bag and also put our ROP chain (which we will pivot to later) in the bag.
Call readin(idpto, what) to write to ptrto.




相关资料：
http://acez.re/ctf-writeup-hitcon-ctf-2014-stkof-or-modern-heap-overflow/
https://github.com/ctfs/write-ups-2014/tree/master/hitcon-ctf-2014/stkof
