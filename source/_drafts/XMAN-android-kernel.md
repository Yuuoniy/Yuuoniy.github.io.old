---
title: 安卓 NativeLib /kernel fuzz,调试和利用
date: 2018-08-16 14:02:10
tags:
---
玄武实验室的刘惠明
xlab@tencent.com

#### 安卓nativelib 和 kernel 简介和攻击面
  ##### 安卓底层安全架构简介
    - android 架构

    - android 底层安全框架及攻击缓释技术
      - 自主访问控制
        基于uid gid 容易被突破
      - 强制访问控制
        - SELunix
      - 验证启动模式
      - 代码签名和平台密钥
      

    - 常用工具 技术 
  ##### 原生代码库及其攻击面
    
  ##### 内核攻击面

#### 常用Fuzz 技术
  ##### alf
  ##### syzkaller
#### 漏洞调试和利用
  ##### buleborne 
  ##### dirtycow


内容
  - 基础知识
  - 原生库漏洞Fuzz和调试利用
  - 内核漏洞Fuzz和利用
