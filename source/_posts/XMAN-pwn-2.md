---
title: XMAN-pwn-2
date: 2018-08-06 08:53:52
tags:
---
#### linux 堆管理 
  program-(free malloc)-heap allocator-(mmap,brk)-kernel
  glic-ptmalloc2
  主要结构：chunk,bin,arena
  bins 管理空闲堆块
  arena 记录分配信息 
  chunk --allocated/free
  malloc 返回指向data的指针。
  - chunk size包含堆头部
  - 对齐问题，size 低三位恒为0 用作标志位
    - N:是否thread arena
    - M:是否mmaped 分配
    - P:前一个chunk 是否适用
  - pre_size:前一个相邻块大小
    - 前一个块free时有意义
    - 前一个块allocated时可以空间复用为数据空间。

  - free chunk
    - fd:freelist中的 
    - bk
    相关数据结构
  - top chunk
  - bins
    - fast bin
      FILO，总共10bins, 每个bin中chunk大小一样 0x20-0x80(64bit) 包括头部的,0x10大小递增
      仍被标记pre inuse位。不会进行合并。实际上最后三个bins并不使用
    - unsorted bin
      管理unsorted chunk。所有fast chunk之外的chunk被free时都会先放入unsorted bin
      只有一个bin，FIFO。bin 中chunk大小不一样。双向循环链表
    - small bins
      0x20-0x400(64bit)0x10递增，62个bin ,每个bin中chunk大小一样 双向循环链表
    - large bins
      双向循环 FIFO。63bins,同一个bin中的chunk大小可以不一样，但在某个范围内。>0x400 用到fd_nextsize,bk_nextsize
  - arana
    - 管理bins。由主线程创建 main_arane,其他线程创建：thread_arena.arena数量有限 32：2*cores+1,64:
    - malloc_state:fastbinY fastbin指针数组 Bins 除了fastbins 的指针数组
  - malloc 的操作：
    - 计算真正的堆块大小：用户管理+堆头 
    - 在fastbin范围内？
      - 是，检查对应的bin是不是有chunk
        - 是，取bin的chunk
    - 在small bin 范围？
      - 是，对应的Bin有chunk
        - 是，取chunk
    - unsorted bin有chunk?
      - 有，从尾部去第一块，大小是否满足分配要求
        - 是，剩余大小是否>minsize(堆块头部大小))
          -是，切分块，返回用户，把剩余chunk放回
        - 从unsorted 取下该块，不会切割
      - 把这个块放到相应的small bin/large bin,遍历下一个块
    - unsorted bin 也不能满足要求
      - 大小是否在large bin
        - 是否有chunk
          - 有,找到满足需求的最小chunk, 切分块(看生于大小）并返回，将剩余的块放入unsorted bin
      - large bin 分配失败
        - 再次浏览所有的small/large bin 找到 best-fit chunk。
    - top chunk
    - 系统调用扩大堆空间
  - free 操作
    - 属于fast bin
      - 是，直接加入对应fast bin，不改变下一堆块pre inuse标志位
    - 是通过mmaped分配吗
      - 是，用munmap回收
    - 前一个相邻chunk是空闲的吗
      - 和前一个堆块合并
    - 后一个相邻chunk是top chunk?
      - 是，和top chunk合并
    - 后一个相邻chunk空闲？
      - 合并
    - 连入unsorted bin

#### 常见堆漏洞
- 堆溢出
  - 修改下一个堆块的size,标志位，fd/bk指针
  - 修改下一个对手的用
- uaf
  - fd bk指针
- double free
  - 绕过glic中的检查。
- unlink:
  unlink
  chunk overlap ：overwrite next chunk's size=>extend allacted chunk
  overwrite next chunk'size=>shrink free chunk
  xx=>extend free chunk
- off by one
  - 改写下一个块的size
  - 改写下一个块的pre inuse位
  chunk overlap ：
  - overwrite next chunk's size=>extend allacted chunk
  - overwrite next chunk'size=>shrink free chunk
  - xx=>extend free chunk
#### 堆利用技巧
##### fastbin attack
  - 控制fd指针
  - size 检查，链表头检查
  - 利用misalignment 伪造=>0x7fxxxx
  - overwrite GOT
  - overwrite main_arena(always top chunk_ptr)
  - overwrite realloc_hook/malloc_hook/free_hook (size 难伪造，but always infeasible)
##### unsorted bin
  - 任意地址写固定值 
  - **控制bk指针**，bk=addr-0x10 ，想要改写的地址值
  - malloc 一个大小正好的堆块
  - 达到效果：addr=main_arena+0x58 ,转化为fastbin
  - 通常改写 global_max_fast
  
##### small bin stack
  - 伪造fake chunk,并让fake chunk 的fd 指向victim
  - 改写victim的bk为fake chunk
  - 则可以吧fake chunk 翻入对应的small bin
  - 下次malloc就可以分配到fake chunk
##### large bin
  - 出现在从unsorted bin 放入large bin的过程
  - 吧large chunk 的 bk_next_size 改写成addr_A
  - 通过malloc 吧一个比当前（不满足
  - addra+0x20 将被改写成victim指针


关键：控制fd指针，指向想要分配的地址值，uaf漏洞直接改写
free(1) free(2) free(1) ,即使malloc 又是 free,就可以改写
伪造chunk p64(0x0)


unlink:
地址 前面伪造fd  bk,-0x18
#### house of xx系列
- **house of spirit**
  - fa
- house of force
  - overwrite top chunk's size
  - malloc arb


- house of lore

- houser of roman 
  没有泄露
  malloc hook 



pie:full relo:不能修改GOT,只能修改hook
- free_hook 
- malloc_hook 改one_gadget          因为参数不能控制
- 利用malloc_printer 实现one_gadget 
  - execve("bin/sh",rsp+50) 用malloc printer(在malloc,free中的错误检查) 函数内部又调用malloc

- 非对齐的方式伪造size
- 利用fast bin 在main arana 写size
- 利用 fast bin 写 main arean 的size？

函数名-检查 表
- 改写关键数据结构
- 跳出 heap 范围
- 构造 overlap chunk

多调试看堆内存怎么分布。

top chunk：
泄露heap起始地址(fast bin的fd 指向另外一个)
泄露Libc: 利用uaf,放入unsorted bin:fd bk写入libc中地址？。从
chunk3的fd指向 chunk4
top chunk的值size 变成system-1
top_ptr+free_hook_top_ptr=free_hook,写入system-1 top chunk标志位最后一位要是1，对执行整个system函数没有影响。 
p &__free_hook__
main_arena <—-> symbols[‘__malloc_hook’]+0x10
note writeup:
note 在栈溢出
利用：利用栈溢出将覆盖v7为ptr指向的地址 ，同时利用一开始读取name，在ptr附近构造fake chunk size为0x70。之后对v7 free时就可以将ptr-0x10的地址加入fastbin。
new 一个长度为0x60的就会分配给他。并且地址是ptr的地址。将atoi_got写在ptr地址处，即ptr[0]=atoi_got通过show,就可以显示ptr[0]指向的内容，也就是atoi的地址。从而计算出libc基址，得到system地址。再通过edit函数修改ptr[0]指向的内容，即atoi_got指向system地址。

https://www.xctf.org.cn/library/details/8629b3304979199d7af6a4ad1bd94274c0a6a32a/


阅读源码
https://bbs.pediy.com/thread-222718.htm

malloc_usable_size
