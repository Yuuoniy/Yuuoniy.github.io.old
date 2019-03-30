---
title:  CTF | 逆向入门第二篇
date: 2018-08-03 08:50:25
tags: reversing
categories: CTF
---
逆向入门第二篇
<!-- more --> 
#### hide
strings 查看到有壳upx 3.91
![](http://p6cwaqxnz.bkt.clouddn.com/upx3.91.png)
##### 脱壳
不能使用upx -d 直接脱。

运行起来相当于脱壳看，因此可以用 dd 把data和text两段dump下来之后拼在一起，可以在IDA里面看。但实际上文件结构会被破坏，因为是运行时镜像，动态脱壳都有这样的问题。所以动态调试还是使用有壳的。dd 
cat /proc/(pid hide)/maps 可以查看进程的内存映射
脱壳命令,其中的地址就是各段的地址,因为是静态的,因此都是不变的，可以直接复制：
```
sudo dd if=/proc/$(pidof hide)/mem of=hide_dump1 skip=4194304  bs=1c count=827392
sudo dd if=/proc/$(pidof hide)/mem of=hide_dump2 skip=7110656  bs=1c count=20480
cat hide_dump1 hide_dump2 >hide_dump
```
看到别人说gdb dump binary 更方便,不过没试过(不会)
##### ptrace反调试 
**ptrace介绍**： ptrace提供了一种使父进程得以监视和控制其它进程的方式，它还能够改变子进程中的寄存器和内核映像，因而可以实现断点调试和系统调用的跟踪。使用ptrace，你可以在用户层拦截和修改系统调用(sys call)，修改它的参数，插入代码给正在运行的程序以及偷窥和篡改进程的寄存器和数据段。

使用strace
**ptrace 相关的反调试** 
需要把反调试过掉
gdb 中 `catch syscall ptrace` 设置断点
可以发现程序在`cmp eax,0` 处断下,此时通过`set $eax=0`,使程序可以往下执行。
c ,再修改一次，就可以成功过掉反调试了>< 队友说,ptrace 大概就是看eax有没有改变?一般就是`cmp eax,0`的地方了
 

######  附gdb 操作
- hb *0x start地址 下硬件断点
- record full (只有在静态执行时有用) 之后可以倒着执行也可以正着执行
- rsi 
- ni
- si 
- reverse-continue 倒着执行回到上一个断点

ELF：init区，在main 执行前会执行
**字符串交叉引用** enter flag 看到哪些地方引用，找到藏起来的代码
p 出来F5有问题,算是IDA的bug吧。

```x86asm
mov eax xxh
syscall
cmp eax 0
jnz xx
```
忽视syscall 会改变eax
```x86asm
xor eax eax
syscall 
cmp eax
```
所以此处也要改
修改之后的：
![](http://p6cwaqxnz.bkt.clouddn.com/hide-jz.png?imageView2/2/w/200)
结果：
![](http://p6cwaqxnz.bkt.clouddn.com/hide-patch.png?imageView2/2/w/200)
jnz 改成jz，就可以看但是语义会改变
继续逆向程序,可以发现是常见算法 Tea
首先会判断字符串长度和几位字符是否符合要求，之后进行加密
加密过程为 xTea->异或->xtea。。
但是对于我来说..看半天都不知道是xtea加密吧
v1 为加密后的flag  
解密获得key  delta xtea  
嗯..开始写脚本了...对我来说好难...还是试一下...先github上找到xtea
和网上找的xtea加密是一模一样的，应该没有修改
注意加密的v0,v1分别取data二进制数据的0-4,和4-8位,所以需要struct
感觉我反汇编还是有问题...看了别人写的脚本，和自己看到的加密有点不一样，尤其是sum部分。
终于...原来是我脚本的问题..
字符串定位找到关键函数  
附：[2018年强网杯初赛 逆向题目 hide writeup (超详细)](https://blog.csdn.net/buaaqqq2015/article/details/79736026)  
**符号匹配**
总结：ida flirt/rizzo.py 比IDA官方好,不支持IDA7.0/ https://github.com/A7um/CTFUtils 支持IDA7.0

写个简易的源码
gcc --static 
安装插件 rizoo (autm改进支持7.0的)
file->produce file->rizzo
file->load file->rizzo
静态编译，得到有符号的，使用工具匹配对照看，produce /load 
windows 程序比较有用
`ps -ef | grep` 进程名


#### re2
hook 原理
https://bbs.pediy.com/thread-228669.htm
可以看出来是修改writefile的前五个字节，但是怎么看出来跳转到哪个函数的？
还需要注意优先级的问题，^ 和 +,不知道的话还是挺坑的...
#### magic 
动态调试更方便  
F5有问题,修改main调用约定  
**调用约定** 
options->complier->编译器->Vistual C++ 就可以了
首先设置断点发现问题，使用trace找到main之前调用的函数，通过交叉调用分析函数。
首先过掉 check_sercet  
(大佬是一开始交叉引用一波,然后看scanf的交叉引用就找到了关键函数)
前三个
最后一个
simple_enc
int __usercall xxx@<rax>

setjmp：设置jmp点。longjmp: 跳到setjmp设置的位置异常处理 
虚拟机逆向题

相关writeup
https://www.52pojie.cn/thread-742361-1-1.html  RCTF - magic
https://4hou.win/wordpress/?p=20989 
haskell 编译中间语言  
oooverflow   
34c3ctf   

了解到虚拟机逆向...我对逆向还真的一无所知  




需要学习的: 
- gdb 使用
- 虚拟机逆向
- 逆向题分析
- 脱壳
- 动态调试

#### 非常规逆向
- lua/python/java/lua-jit/haskell/applescript/js混淆方法多/solidity/webassembly/etc
- firmware固件/raw bin/etc
- chip8/avr/clemency/risc-v/etc

##### 找工具
- binary parser
- disassembler 
    -必要的
- tracer
    - disassembler+tracer=debugger
- debugger
- decomplier

##### 格式
读文档  
工具链 toolchains
教程 tutorial
例子 
 - haskell


##### find binary
- google
- firmware
    + rebase the binary 还原基址
    + recover the symbol table 
- other
-  -strings/binwalk
-  IDA/radare2/binary.ninja/ida loader

##### disassembler
- google 'xxx dissemab/xx IDA '
    -avr ida
- 人工
- ida pro/radare2/binary.ninja interface
    IDA processor

##### tracer
- google 先搜文档
- tracer
- debugger
    -  gdb-mutiarch
    -  qemu
    -  emulator
    -  trace reply
- 例子
    - solidity

##### 逆向工程
find code pattern

#### printf_machine
实现一套指令集
mov语句,脚本翻译
符号执行 

#### assem
写个html 在浏览器调

#### re_3
ida/procs/
https://github.com/themadinventor/ida-xtensa 
上网找适配器
找flag 字符串交叉引用

#### easy
haskell 逆向
有反汇编 网上找

#### IDA 
#####  断点
* 执行断点是软件断点。 
* 内存访问断点是硬件断点。

##### trace 功能


- 指令跟踪：IDA将会记录每一条指令的执行，并保存寄存器数值，通过使用这些信息，你可以找出应用程序的执行过程，并可找出哪条指令修改了哪个寄存器。

- 函数跟踪：IDA将会记录所有的函数调用和函数返回。

- 读写-写-执行跟踪：IDA将会记录一个对指定地址的所有访问。这种机制相当于是不停止的断点。

对每种跟踪机制，都会记录相应的跟踪事件到跟踪缓存区，也可以保存到一个txt的文件中，同样可以通过Tracing-options里的选项来设定。

##### 插件：
keypatch
findcrypt-yara 找加密方式的插件 https://www.cnblogs.com/zhaijiahui/p/7978897.html
FRIEND 链接：https://github.com/alexhude/FRIEND

 翻页 esc 和 Ctrl+Enter 

**强制类型转换**


P:还是懒得贴图，我以后不知道自己在写什么鬼东西时一定会后悔的 嗯...
大佬讲的是在太难了,我还是先去做一些基础题呜呜呜
