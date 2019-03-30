---
title: CTF | ida 基础
date: 2018-09-21 12:35:13
tags:
---
介绍 IDA 基本使用
<!-- more -->
一些名称

```
sub_xxxxxx： 地址xxxxxx处的子例程
loc_xxxxxx：地址xxxxxx处的一个指令
byte_xxxxxx：位置xxxxxx处的8位数据
word_xxxxxx：位置xxxxxx处的16位数据
dword_xxxxxx：位置xxxxxx处的32位数据
unk_xxxxxx：位置xxxxxx处的大小未知的数据
```

使用：
C ：在指定代码行进行操作，IDA将尝试反编译所有字节为指令。（常用于反编译时，遇到DCB…什么等数据）
D ：在指定代码行进行操作，IDA将指令批量转换为数据。
