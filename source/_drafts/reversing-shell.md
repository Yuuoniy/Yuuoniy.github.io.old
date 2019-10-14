---
title: reversing-shell
date: 2018-08-15 18:54:32
tags: reverse
---

OEP:original entry point 原始入口
- pushad 压栈，代表程序入口点
- popad 出栈，代表程序出口点，一般找到这个OEP就在附近?

- 单步追踪法
  . 像这种入口点附近就是一个call的我们称为近call, 对于近call我们选择步进, 按下F7(当然你也只能选择步进, 不然EIP就跑偏程序停止了)
 
- ESP 定律

- 花指令
