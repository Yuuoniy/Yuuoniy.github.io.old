---
title:  CTF | sysucsa 2018 招新赛 pwn writeup
date: 2018-09-24 11:26:17
tags:
---

### ssp
在加了canary保护之后函数开始时，会取gs:0x14处的值，并放在%ebp-0xc的地方(mov %gs:0x14,%eax, mov %eax,-0xc(%ebp))，在程序结束时，会将该值取出，并与gs:0x14的值进行抑或(mov -0xc(%ebp),%eax，xor %gs:0x14,%eax)，如果抑或的结果为0，说明canary未被修改，程序会正常结束，反之如果抑或结果不为0，说明canary已经被非法修改，存在攻击行为，此时程序流程会走到__stack_chk_fail，从而终止程序。


利用方式：
从Canary的工作机制，可以总结出绕过Canary保护的方法有：

1.泄露canary。由于Canary保护仅仅是检查canary是否被改写，而不会检查其他栈内容，因此如果攻击者能够泄露出canary的值，便可以在构造攻击负载时填充正确的canary，从而绕过canary检查，达到实施攻击的目的。NJCTF2017的messager就是通过这个方法利用。

2.劫持stack_chk_fail。当canary被改写时，程序执行流会走到stack_chk_fail函数，如果攻击者可以劫持该函数，便能够改变程序的执行逻辑，执行攻击者构造的代码。我们知道，Linux采用的是延迟绑定技术(PLT)，如果我们能够修改全局偏移表(GOT)中存储的__stack_chk_fail函数地址，便可以在触发canary检查失败时，跳转到指定的地址继续执行。

3.利用canary在发生栈溢出是的警告信息，需要对__stack_chk_fail函数的执行流程有一定了解，在错误提示中，会将你发生栈溢出的程序名调用输出，其位置位于argv[0]，我们可以将argv[0]的地址改写为我们想要获取的内容的地址，使它随着错误提示一起输出。
参考链接：http://yunnigu.dropsec.xyz/2017/03/20/Liunx%E4%B8%8B%E5%85%B3%E4%BA%8E%E7%BB%95%E8%BF%87cancry%E4%BF%9D%E6%8A%A4%E6%80%BB%E7%BB%93/

学习如何绕过 canary:https://0x48.pw/2017/03/14/0x2d/

源码：https://code.woboq.org/userspace/glibc/debug/fortify_fail.c.html#__fortify_fail_abort


```c
__stack_chk_fail (void)
{
  __fortify_fail_abort (false, "stack smashing detected");
}
```

```
__fortify_fail_abort (_Bool need_backtrace, const char *msg)
{
  /* The loop is added only to keep gcc happy.  Don't pass down
     __libc_argv[0] if we aren't doing backtrace since __libc_argv[0]
     may point to the corrupted stack.  */
  while (1)
    __libc_message (need_backtrace ? (do_abort | do_backtrace) : do_abort,
                    "*** %s ***: %s terminated\n",
                    msg,
                    (need_backtrace && __libc_argv[0] != NULL
                     ? __libc_argv[0] : "<unknown>"));
}
```
	
