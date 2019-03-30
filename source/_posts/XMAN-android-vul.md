---
title: XMAN-android-vul
date: 2018-08-13 09:06:15
tags:
---
- 内部存储
  - 模式
  - 读文件
- 读文件
- 缓存文件

- 外部存储
  - 获取权限
  - getE

- 应用私有目录
  - /data/data/appname
    - databases:数据库
    - cache 缓存数据
    - files 自己控制的文件 
- sharePreference
 - 以xml形式 


- 应用沙箱
  - root 用户UID为0
  - 系统服务UID从1000开始，system 用户UID为1000，应用程序UID从10000开始到19999
  - 同签名应用程序可以使用

- 权限
  - 

- 权限保护级别
  - Normal
  - dangerous
  - signature
  - signatureOrSystem

- SERvice
  - startService
  - bindService
  - intentService
    - 可以执行长时间逻辑漏洞的
    - 所有请求处理完成后，intentservice 自动结束

- contentProvider
  - 不同app之间交换数据的标准api
  - 

- Broadcast
  - 跨进程的消息收发机制
  - sendbroadcast
  - 继承BroadcastReceiver,重写onReceive
  - 在manifest.xml中注册

  - 注册方式
    - 静态注册
    - 动态注册

- intent
  - 显示intent
  - 隐式intent

- webview
  - setSavePassword
  - setJavascriptEnabled
  - setAllowFilAccess
  - 

- 常用分析方法及工具
  - 流量 HTTP/HTTPS
    - brup suit
    - charles
    - fiddler
  - 其他
    - tcpdump
    - wirecharsk


filetree
- 动态调试  

-  风险
  - allow backpup
  - cl
  

- 漏洞
  - MIMT 不安全的HTTPS
    - trustManager
    - hostnameVerifier
    - 通过inetAddress作为createSocket参数

  - WebView
    - 未移除隐藏的Webview接口
    - 在实现webview直接把接口移除掉
    - webview跨域信息泄露
      - 开启JS支持
      - 允许使用file协议
      - 未对file协议作检查

  - SQL
  - web api

  - 越权 
  - 路径穿越   用来作为组合漏洞进行攻击 
       https://www.tr0y.wang/2018/05/15/zipperdown/
       https://mabin004.github.io/2018/03/30/zip%E8%A7%A3%E5%8E%8B%E6%BC%8F%E6%B4%9E/
  - 私有数据泄露 contentProvider  
    http://jaq.alibaba.com/blog.htm?id=61 

  - 本地开放端口

  - DOS 
    Java层
      - 没有对异常进行恰当处理
    native层
      - 二进制漏洞
      - 没有恰当的异常处理
  - 二进制漏洞

  自动化工具
  - qark
  - mobile-security-framework-mobsf
  - janus
  - flowdroid
  - drozer




