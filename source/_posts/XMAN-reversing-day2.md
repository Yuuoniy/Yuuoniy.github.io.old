---
title: XMAN-reversing-day2
date: 2018-08-03 08:50:25
tags: xman,reversing
---
逆向入门第二篇
<!-- more --> 
#### hide
strings 查看到有壳upx 3.91


[2018年强网杯初赛 逆向题目 hide writeup (超详细)](https://blog.csdn.net/buaaqqq2015/article/details/79736026)

运行起来相当于脱壳看，因此可以用dd 把两段(cat /proc/pidof xx/maps)dump下来之后拼在一起，可以在IDA里面看。但实际上文件结构会被破坏，因为是运行时镜像，动态脱壳都有这样的问题。dd 
有反调试 
使用strace
ptrace 相关的反调试 
需要把反调试过掉
set $eax=0
gdb:
catch syscall ptrace 


main 函数：
 if ** 反调试的位置
 write
 read
 check_flag strcmp

 把代码藏起来了
 gdb 
 catch
 hb *0x start地址 下硬件断点
 record full (只有在静态执行时有用) 之后可以导着执行也可以正着执行
 rsi 
 ni
 ni
si 
reverse-continue 倒着执行回到上一个断点

ELF：init区，
字符串交叉引用 enter flag 看到哪些地方引用，找到藏起来的代码
p 出来F5有问题。
mov eax xxh
syscall
cmp eax 0
jnz xx
忽视syscall 会改变eax

xor eax eax
syscall 
cmp eax
所以此处也要改

jnz 改成jz，就可以看但是语义会改变
是常见算法 Tea 
Tea->异或->tea。。
v1 为加密后的flag
解密获得key  delta xtea

字符串定位找到关键函数
**符号匹配**
总结：ida flirt/rizzo.py 比IDA官方好,不支持IDA7.0/ https://github.com/A7um/CTFUtils 支持IDA7.0

写个简易的源码
gcc --static 
安装插件rizoo (autm改进支持7.0的)
file->produce file->rizzo
file->load file->rizzo
静态编译，得到有符号的，使用工具匹配对照看，produce /load 
windows 程序比较有用
ps -ef | grep 进程名

#### magic 
动态调试更方便
F5有问题,修改main调用约定
**调用约定**
options->complier->编译器->Vistual C++ 就可以了
首先设置断点发现问题，使用trace找到main之前调用的函数，通过交叉调用分析函数。
首先过掉 check_sercet



9 
前三个
最后一个
simple_enc
int __usercall xxx@<rax>

setjmp：设置jmp点。longjmp: 跳到setjmp设置的位置异常处理 
虚拟机逆向题

相关writeup
https://www.52pojie.cn/thread-742361-1-1.html  RCTF - magic
http://wemedia.ifeng.com/65490822/wemedia.shtml 详细
https://4hou.win/wordpress/?p=20989 
haskell 编译中间语言
oooverflow 
34c3ctf

了解到虚拟机逆向...我对逆向还真的一无所知
还有一堆爆破随机数种子blabla。。逆向逻辑对我来说就好难
不过真的很有趣啦！自己也愿意学 要是有更多时间就好了


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

#### 找工具
- binary parser
- disassembler 
    -必要的
- tracer
    - disassembler+tracer=debugger
- debugger
- decomplier

#### 格式
读文档  
工具链 toolchains
教程 tutorial
例子 
 - haskell


#### find binary
- google
- firmware
    + rebase the binary 还原基址
    + recover the symbol table 
- other
-  -strings/binwalk
-  IDA/radare2/binary.ninja/ida loader

#### disassembler
-google 'xxx dissemab/xx IDA '
    -avr ida
-人工
-ida pro/radare2/binary.ninja interface
    IDA processor

#### tracer
- google 先搜文档
- tracer
- debugger
    -  gdb-mutiarch
    -  qemu
    -  emulator
    -  trace reply
- 例子
    - solidity

#### 逆向工程
find code pattern

### printf_machine
实现一套指令集
mov语句,脚本翻译
符号执行 

### assem
写个html 在浏览器调

### re_3
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

插件：
keypatch
findcrypt-yara 找加密方式的插件 https://www.cnblogs.com/zhaijiahui/p/7978897.html
FRIEND 链接：https://github.com/alexhude/FRIEND

 翻页 esc 和 Ctrl+Enter 

**强制类型转换**
#### 常见函数
time64
