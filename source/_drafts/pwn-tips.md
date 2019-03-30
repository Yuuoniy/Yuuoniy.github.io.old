---
title: CTF | pwn-tips
date: 2018-09-06 17:17:32
tags: 
---
记录 ctf 中 pwn的小技巧
<!-- more -->
pwndbg 计算偏移：
`cyclic 200` 生成字串
`cyclic -l xxxx` 查找



各种安全选择的编译参数如下：

- `NX：-z execstack / -z noexecstack` (关闭 / 开启)
- `Canary：-fno-stack-protector /-fstack-protector / -fstack-protector-all` (关闭 / 开启 / 全开启)
- `PIE：-no-pie / -pie` (关闭 / 开启)
- `RELRO：-z norelro / -z lazy / -z now` (关闭 / 部分开启 / 完全开启)

NX:堆栈不可执行
Canary:栈溢出保护
PIE：地址随机化
RELRO：不能修改GOT

gcc 编译32位程序 安装：
```
sudo apt-get install build-essential module-assistant
sudo apt-get install gcc-multilib g++-multilib
```

x/4wx
x/s
x/i

### 小工具
- roputils 
[github](https://github.com/inaz2/roputils/blob/master/README.md)
- [ropper](https://github.com/sashs/Ropper)

### `tmux` 终端管理
快捷键
上下分屏：`ctrl + b`  再按 "

左右分屏：`ctrl + b`  再按 %

切换屏幕：`ctrl + b`  再按 o

关闭一个终端：`ctrl + b`  再按x

上下分屏与左右分屏切换： `ctrl + b`  再按空格键
https://github.com/gpakosz/.tmux
prefix + H J K L 调整大小
开启鼠标支持
$ tmux set -g mouse on

鼠标支持默认是关闭的，开启鼠标后，支持复制，翻屏，切换面板，切换窗口，resize。
### 配置带符号信息的libc
首先安装
`sudo apt-get install libc6-dbg`
添加源 sudo apt update
然后下载源码 sudo apt source libc6-dev
记下我们源码的目录：之后在 gdb 中就可以 directory ~/glibc-2.24/malloc/ 
在开始调试之前，需要指定一下刚刚获得的带有libc6-dev源码文件夹的路径，比如我把这些源码放在了 ~/glibc/lib 文件夹下，通常一般程序需要的是stdio-common这个目录内的文件，
我的路径是 ~/glibc-2.23/
然后就可以啦！
于是输入 `(gdb) directory ~/glibc/lib/stdio-common`

gdb 程序名

one gadgets rce
With one-gadget-RCE, we can just hijack .got.plt or something we can use to control eip to make program jump to one-gadget, but there are some constraints that need satisfying before using it.


## 不同利用方式：

### Hijack hook function
条件：
1. 泄露libc地址
2. 任意写
3. 调用malloc,free 或者 realloc

__malloc_hook
__free_hook

Since hook function is called in the libc, the constraints of one-gadget are usually satisfied.

### Use printf to trigger malloc and free

### execveat 起 shell

```c
int execveat(int dirfd, const char *pathname,
             char *const argv[], char *const envp[],
             int flags);
```

x86 是 32 位，x64 是 64 位。

### 调用 main 函数：
```
push envp 
push argv 
push argc 
call main 
...
```
即使你将main()声明为不带参数的main()函数。它们仍然在栈中,只是没被使用。

### 传参方式
在x86中,最常见的传参方式是 `cdecl` :
```x86asm
push arg3
push arg2
push arg1
call f
add esp, 12 ; 4*3=12
```
`syscall(rax,rdi,rsi,rdx) `
`syscall(rax,rdi,rsi,rdx) `

 64 位泄露地址需要把 debug 位置为 1？

 ### puts
 遇到 '\x00' 截断，会添加回车
 ```python
 data = io.recv()[:-1]   # 去掉末尾\n
if not data:
    data = '\x00'
else:
    data = data[:4]
```

### printf
遇到 '\x00' 会截断，不会添加回车
为了防止 printf 的 %s 被 \x00 截断，需要对格式化字符串做一些改变


http://ropshell.com/

http://0xpoker.cuit.site/20171206/ROP%E5%AD%A6%E4%B9%A0%E4%B9%8B%E6%97%85(%E4%B8%80)/


64 位程序的前六个参数通过 `RDI、RSI、RDX、RCX、R8 和 R9` 传递。4 位可以使用的内存地址不能大于 0x00007fffffffffff，否则就会抛出异常。

fastbin attack
-  **Arbitrary Alloc**
分配到 malloc_hook 附近，利用字节错位等方法来绕过 size 域的检验，实现任意地址分配 chunk

- 
常见利用思路：
基本利用思路如下

1. 利用 unsorted bin 地址泄漏 libc 基地址
2. 利用 double free 构造 fastbin 循环链表
3. 分配 chunk 到 malloc_hook 附近，修改 malloc_hook 为 one_gadget

编译时如果加上“-finstrument-functions”选项，那在每个函数的入口和出口处会各增加一个额外的hook函数的调用
```
void __cyg_profile_func_enter (void *this_fn, void *call_site);
void __cyg_profile_func_exit  (void *this_fn, void *call_site);
```
https://www.anquanke.com/post/id/86286

此题程序功能可看成是模拟协议数据单元（PDU）的传输与重组

小工具：main_arena_offset
https://github.com/bash-c/main_arena_offset
使用：
```c
yuuoniy@yuuoniy-virtual-machine:~$ main_arena /lib/x86_64-linux-gnu/libc-2.23.so
[+]__malloc_hook_offset : 0x3c4b10
[+]main_arena_offset : 0x3c4b20
```

one_gadget:
 one_gadget  /lib/x86_64-linux-gnu/libc-2.23.so

系统调用号
32位
`cat /usr/include/asm/unistd_32.h`

64位
`cat /usr/include/asm/unistd_64.h`

#### core dump
`ulimit -c`
`ulimit -c unlimited` 开启
`gdb program core`

fastbin 大小：
在32位系统中，`fastbin` 里 `chunk` 的大小范围从 `0x10` 到 `0x40` ；在64位系统中，fastbin 里 chunk 的大小范围从 `0x20` 到 `0x80` 。

unsorted bin 从头部插入，从尾部取出。

### tmux
ctrl+b +方向键：窗体之间切换
ctrl b +数字 窗口切换
Ctrl-b + [ 进入滚动模式


malloc常看源码：https://code.woboq.org/userspace/glibc/malloc/malloc.c.html#__malloc_hook

visual stdio 快捷键：ctrl+shifit +v :markdown 预览
ctrl+shift+c 当前目录打开cmd
