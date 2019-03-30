---
title: XMAN-pwn-4
date: 2018-08-08 08:48:35
tags:
---
### basic
1. some tools
  - ida pro
  - x64 dbg
  - windbg:kernel
  - Mona:pytho 脚本 kernel调试比较有用
2. pe 文件
  pe header 
  import address table
  export address table
  section table

  optional header:IAT导入表 EAT导出表

pe memory layout
anything.exe
常见dll文件：
- msvcrtxx.dll (libc):c 语言库
- kernel32.dll:create file api
- ntdll.dll:native api
- user32.dll:message box 

内存布局
kernel memory/various dll/your

常见漏洞：
nutshell
栈溢出
buffer|canary| eip| seh|:linux 没有，catch 处理expction 结构化异常处理
seh 可以 bypass 
seh:单链表，pre hadler->oxffff handler 在栈上
只能控制一次EIP，栈 想办法跳到控制的内存

mov eax,dword ptr [ebx+xx]/mov eax,dword ptr [ebx+0xx]



19f6b0 栈顶，控制的buffer为第二个参数，
https://blog.csdn.net/hustd10/article/details/51167971 解释为什么控制第二个参数
第二个参数EstablisherFrame，这个参数实际上就是当前SEH链的首地址，即SEH链中第一个_EXCEPTION_REGISTRATION_RECORD结构的地址，也就是被我们改写的那个异常处理器的地址。当异常处理函数被调用的时候，这个参数位于栈上ESP+8的位置。这不难理解，这个函数为__cdecl调用方式，则栈上是这样子的：  

add esp 0xxxx
ret 
19ff70 指向能控制的buffer 算好偏移，跳到控制的

pop xxx;pop xxx;pop esp;whaterver ;ret(就是把19ff70前面的pop掉，*将19ff70 pop到esp中*。)
seh 是盖过，控制EIP，控制
只能控制pointer指向buffer的东西，指向的地方可以写shellcode,
IDA 看不出catch 会把后门放这里

mov dword ptr fs:[0],aaa
注册异常处理,fs段选择子？存储 seh 第一个结构的地址 64位不在这里

加花？木马免杀
mov dword ptr fs:[0],0xadcd
xor eax,eax
mov dword ptr [eax],1234
崩掉，进入异常
0xabcd 为木马入口点，触发一个异常


fs:[0] teb 线程的环境块 
_security _cookie =canry
eax 写的时候崩掉
linker-advance 把保护去掉

{
  char *p=0
  __try{
    *p=0;
  }
  __except(1){
    puts("hi ex);
  }
}
execve()
windows:
system("cmd.exe")-->msvcrtxx.dll(libc) xx代表数字，可能编译对方的机器没有这个dll

WinExc("cmd.exe",SW_SHOW(5))  
CreateProcess  参数很多. kernel32.dll
--->NtCreateProcess->ntdll 
NtCreateProcess 和 execve 不一样，NtCreateProcess创建成功，原来的进程也在，execve返回仅在调用错误的时候
system("cmd.exe") exit(0)/ ExitProcess(0)

64为seh不在栈上。
nc -e stack -p xxx -l


checksec
PEID
detect  it easy

32位fs
64为gs：gs里没有exception?
导入表是只读的，不能修改。和got不一样

stack:
- leak crucial add address
- control EIP again
- call shell
两次rop ,1.leak 2.get shell
三种方法：
SEH overflow in x64 不行，因为不在栈上。
/gs 编译的参数，canary

chunk 的 size 加密，key 在 teb ？随机 的
- HEAP
LFH: 
VS:很大程度随机化分配
超过连续十八次..会自动开启LFH
大于16k.VS分配？
vs 分配和linux差不多

- encoding,随机的key加密
- low fragment heap
  默认开启LFH
- guard pages
  两个segement之间，

NtVirtualAlloc api kernel32-->mmap 
NtAllocaVirtualMemory -->mmap
分配两个大块，中间有guard pages

**LFH bypass**
堆的随机化
只能躲
- 十八次以内
- 大于16k
- heap spary 堆分？
- low fragement heap 

覆盖虚表
覆盖后看eax变化
thiscall --> ecx 就是class本身。ecx一定是this指针
this ecx-->buffer xchg esp,ecx ret;mov esp,ecx (把ecx放到esp中)，也可以用到esi和ecx是一样的 栈迁移
vtable 栈迁移
迁移后esp指向 heap
stack>heap
winExex / system ->>>malloc ->stack
->> heap mov dword ptr [heap],xxx
所以栈迁移
如果ROP够长->virtualProtext/mprotect->jmp heap 变成可执行的
heap->PEB (进程环境块) ->pid linklist(包含所有的dll  dllname dllbase) 
getProcessHeap->  PEB->heap 想办法覆盖PEB->heap，这样malloc时就不会分配到原来的堆，栈就不会出问题

或者找
pop ebx
xxx
pop eax
xxx

gadget:mov dword ptr [eax],ebx;ret
dword ->5* dword
.data >ROP 
xchg somehow-> esp->.data heap->
win64  

**dword fs:[0] TEB**


object 一开始是vtable,05 当前字节长度 0f 最大长度
覆盖 eax:指针的指针
win64 rcx rdx r8 r9,           
创建chunk,free,再创建，覆盖虚表 崩溃

c++ string internal 实现，修改指针为我们的，输出就可以leak了
把vtable的值覆盖成堆地址，第二个地方的值为data段 ，
5aca50 指向堆 可控的 leak
alt+a attach 
http://homes.sice.indiana.edu/yh33/Teaching/I433-2016/lec13-HeapAttacks.pdf

Windows:都有PIE
重新加载一次地址不会变，linux汇编
DEP=NX

ASLR ：
safeseh:调用seh时，会检查handler是不是合法函数。编译器一开始会有一个表，只读 存储所有合法的。
sehop:保证handler地址不能在栈上
call 地址必须在合法的函数开头

ntdll!NtContinue -> rcx CFG是个合法,需要先泄露ntdll
Context:rip rax rbx rcx rsp
sigretrun 
rip rsp->rop 
thiscall->rcx->object
https://introspelliam.github.io/2017/08/05/pwn/XMAN%E4%B9%8B%E6%97%85-windows-exploit-technique/ 
https://whereisk0shl.top/hitb_gsec_ctf_babyshellcode_writeup.html
http://www.cnblogs.com/flycat-2016/p/5426910.html
http://www.cnblogs.com/flycat-2016/p/5426910.html

镜像劫持
