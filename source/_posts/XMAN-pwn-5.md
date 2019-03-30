---
title: XMAN-pwn-5
date: 2018-08-09 08:54:02
tags:
---
### 漏洞挖掘
bestwing.me 
  代码审计：google v8

  逆向： adobe pdf(muhe)
    静态分析，动态分析
  fuzzing: 插桩 ..
    gdb -q
  - 数组越界：栈溢出 oow
  - 缓冲区溢出
  - 整型溢出->缓冲区溢出
  - 格式化字符串漏洞
  - 指针操作不当 double free/uaf
  - 条件竞争：脏牛
  - 类型混淆：浏览器
  - 逻辑漏洞
1. data-flow 分析
  程序点 CFG图
  程序状态
  分类
  - 流不敏感分析
  - 流敏感分析
  - 路径敏感分析
  分类
  - 过程内分析
  - 过程间分析

2. 符号执行
  程序点
  angr

  BV 中间语言的对象
  z3 作约束求解
  angrop
  patcherex
  driller
  retdec mips  windows
3. angr:
  AEG:
4. pin
pintools
插桩
afl
源代码插桩

5. fuzzing
6. ...
  ipython
sudo apt-get install libtool-bin

https://firmianay.gitbooks.io/ctf-all-in-one/doc/5.1_fuzzing.html
使用qemu 模式
https://github.com/shellphish/fuzzer
zeratools
fuzzer

ida python
检测栈溢出的脚本
angr 7.7.9.21 
### angr 学习：
project.factory 提供了很多类对⼆进制⽂件进⾏分析，它提供了⼏个⽅便的构造函数 
https://docs.angr.io/docs/loading.html angr 文档
