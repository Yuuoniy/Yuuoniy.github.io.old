---
title: CTF | android 安全概述
date: 2018-08-04 08:48:24
tags: XMAN android
categories: CTF
---

###  android 安全概述
<!-- #### android 背景介绍 -->
基于Linux开源操作系统   
adb：Android Debug Bridge  
art  
##### 整体架构
###### 应用程序层
- `java`为主
- 支持C/c++
###### framework 层
- 为应用程序开发提供API
###### android runtime
- 提供了JAVA编程语言核心库的大多数功能
- 应用程序都对应独立的虚拟机
    + `android 5.0`以前：`dalvik` 虚拟机
    + `android runtime`
###### 系统运行库
- C/C++库，为开发者提供服务
    + webkit、sqlite
手机端沙箱还没做好
###### 硬件抽象层
- `android framework` 与`linux  kernel`分隔
- 保护厂商利益，因为linux GPL 协议
- 驱动分为`user space`和`kernel space`
    
###### linux kernel
- 硬件驱动、网络、文件系统...


#### android 安全模型
###### 基于linux用户隔离
- linux:用户 uid
- android:应用程序  uid/父进程一样,先从父进程fork
- 应用有自己的数据存放目录
- 共享uid、编译时要分配好
例子：
u0_a52:user id 为0(不同于uid),a52:10052(uid) 10000+52  
###### 应用权限
- 细粒度控制(网络、硬件、敏感数据)
- manifest.xml 申请权限（android 6可以动态申请）
- 权限检查，andorid 或内核进行
    - check permission api
    - 看权限服务由谁提供

###### 代码签名
- apk 开发者签名
    
###### selinux
- security enhancements for android
- 强制访问控制（MAC）
    - 假root
- 核心系统守护进程和用户应用隔离到不同domain

###### 系统更新(刷机)
- 解锁 bootloader recovery
- recovery 单独小型system 访问所有所有分区，即可以修改
- 只允许签名验证通过的刷写
- 解锁：允许替换recovery 和系统镜像
    - 清除所有用户数据,否则会可能恶意访问用户隐私
    - 例子：三星knotes?，关掉敏感操作 如pay




#### 分析环境与常用工具
- santoku
    + 基于`linux`的`android`取证、逆向、开发的平台
- android studio 
    + 应用开发
    + APK 分析、性能分析
- jeb
    + 逆向工具 
    + 无源码调试
    + 提供`ARM、Mips、Intel x86/64` 反编译支持
    + 插件支持
    + java 版本问题，可以修改`jeb`配置文件
- jadx
    + dex 文件反编译
- smali、baksmali、apktool
    + 反汇编常用工具
    + dex(字节码) 与 smali 代码相互转换
    + apk 文件解包和打包

- ida pro
    + 支持 dalvik(基于栈的虚拟机) 指令集反汇编
    + 动态调试native代码
- 服务器
    + 直接编译android操作系统
    + coding
        * make -j64 与 make -j4区别
符号执行

安卓堆分配机制是另外一套

#### android代码分析与对抗
动态调试：修改android debuggable 重打包
##### 正向编译流程
- 源码-资源文件-AIDL 文件? 依赖打包 -----编译器-生成 dex -debug/release keystore -------打包 生成apk
- APK文件结构
    + zip 压缩包
    + 常见文件
        * manifest.xml 整体描述，组件，权限
        * META-INF 签名信息
        * classes.dex 可执行文件,多个 /dalvik 支持的方法数有限
        - res 可编译资源
        - assets 原生资源，很多加壳代码放在assets和lib 
        - lib 目录：C/C++代码编译so 文件，native逆向重点关注

##### 反编译
- dex 文件
    + java class 优化 java class:常量池 dex 包含所有的class:header、数据结构索引区 逐级索引、data
    + android 可执行文件
    - 包含 dalvik 指令集
- smali 代码学习
    - 网上找资料
    - 每个class都是一个文件
    L：是一个类
    <init> 类的构造函数
    invoke 相当于call
    Z:bool
    寄存器的表示p 参数,v
    内部类

bytecode
jeb 插件 scripts.txt
    快捷键 Q 
jadx ：源码级别的搜索
- JNI
    + java 与 native 沟通的桥梁
    + JNI注册
        * 静态注册 名字对应
        * 动态注册 可以比较好隐藏注册了什么函数 方便加防御措施、混淆
    - **JNI_Onload**
        + 在加载so文件时将被执行,一般看这里有没有问题
ida 中修改为 JNIEev*
- 反调试
    - ida 端口 23946
    - ptrace
    - 时间
    - 断点


JNI 层利用反射机制？？调用java代码   
在native 动态加载处理加壳  
installDexs   
mCookie dex文件特征值array（java,dex 加载的内存机制（native
native c++ 代码  

- demo hiddendata

#### 加壳技术
native 层动态代码加载

- 提供的动态加载
    + dexclassloader:apk/jar/dex SD卡 通用
    + pathclassloader：只能加载安装到系统中（/data/app 的apk 文件
    demo crakeme

先找到`oncreate`
- 加载立即删除
- 不落地的dex加载
- 加载过程更加 隐蔽
- 自己加载逻辑

- android debug bridge
- JDWP java调试栈协议

- 动态调试条件
    - android:debuggable="true"
    - android os
        + ro.debuggable=1
        + boot.img->default.prop 解压手机中img,修改文件
        + 模拟器或`userdebug`编译 
        + mprop、BDOpner(Xposed模块)  从内存中修改属性值

- 无源码动态调试 java
    + smalidea+android studio
    + ida pro 
    + jeb

- native 代码调试
- 动态调试解决`hiddendata`(部分操作步骤见结尾)

##### 方法跟踪
- android 带有`method trace`
    + DDMS
    + cpu Profile
函数调用，CPU耗时

as:tools-andorid devices monitor

##### 脱壳技术
- 运行时机
    + dex 文件加载
    + 类加载
    + 方法执行
- 针对内存加密壳的脱壳技术
     -暴力内存搜索 针对dex内存加载阶段
    - 关键函数断点 dex/类加载关键api
    - DexHunter 开源 通用脱壳技术
    - AppSpear 通用脱壳 
Dex 文件基本格式 
在内存中连续

- 暴力内存搜索
    + dex 文件在内存中有特征
        * dexHunter magic number
        * dex.035 035为版本号，还有037 038
        * dey.036(odex 头部)
- 关键函数代码
    + **dvmDexFileOpenPartial** (Dalvik虚拟机):位于dex优化的地方 dvm.so 文件
        * addr -dex 起始地址
        * len -dex 长度
        * 演示 jscrack 脱壳版本要求4.4.4
            - ida -debugger-attach to process
            - 断点，查看顶部寄存器的两个值 就可以dump出文件
            - 
            ``` python
            from idc import *
            from idaapi import * 
            r0=get_reg_value('r0')
            r1=get_reg_value('r1')
            with open('dump,dex',wb) as fp:
                RefreshDebuggerMemory()
                data = GetManyBytes(r0,r1)
                fp.write(data)
            ```

    + dex 加载流程
        * 验证 dex
        * 生成 odex
        * 解析 odex 生成内存结构
    - 生成`odex`时
        + `dvmOptimizeDexFile`
        + 执行`/bin/dexopt` 进行dex优化 ->dvmCon 
    - DexFile:DexFile && OpenAndReadMagic(ART) 
- **DexHunter** (推到类加载执行完后在dump)与`AppSpear`
    + 主动初始化所有的类
    + dex 文件内存重构技术 AppSpear

##### 反混淆
- 花指令
    + simplify
        * 动态执行去除花指令
    符号执行 suit 框架?
- 名称混淆 
    + 根据类型信息重命名 smali 文件
    + Demo 360 renameByType 脚本修改
        * jeb:文件-scripts-执行脚本
Hook:
- hook 一切（xposed、Frida)
- hook 自己(virtualXposed 双开逻辑、legend) 代码热更新

#### 应用层与Root安全
四大组件
- Activity
- service
- content provider
- broadcast receiver

- 组件在Manifest 注册
    broadcastReceiver 也可以动态注册
- 组件生命周期
    + 
- 组件间通信
    intent为载体，底层由binder(linux kernel)实现

##### 应用层安全
- 内层破坏漏洞
    + 底层服务
        * 蓝牙、V8、binder
- 应用逻辑漏洞
    + 空指针异常
    + 权限设置(组建权限，存储权限)
    + 支付流程
- 网络通信
    + SSL、web 安全 h5直接渲染

- 权限控制
    - intent-filter 导出属性默认为true, android:exported='true'

- intent Scheme URLs: 
    - browser->intent->Activity
    - 加载任意网页

- `addJavascriptInterface`
   - js调用java代码 java代码可以反射
    
    **JAVA反射机制** 是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

- 数据存储
    - /data/data/<package_name>

##### root
- 工程机root
    - adb 以 root 模式启动，随后drop权限
    - adb root 始终root权限
    - su命名 set uid

- 普通机器
    - 主动root superSU
    - 被动root 漏洞一把梭

- superSU
    + 直接刷入su
        * system 分区以nosetuid挂载
        * zygote 被drop权限
        * SELinux 下划分了安全域
    - superSU 架构
        + daemeonsu <-> UNIX socket <->untrusted App root权限守护进程

**apktool**使用   
`apktool d test.apk`
`apktool b test`

**动态调试**   
连接mumu模拟器：
`adb connect 127.0.0.1:7555`
安装apk 
`adb install` 本地apk路径
`adb shell am start -D -n com.j.hiddendata/.MainAcitivty` 调试模式启动一个进程

`ddms：android studio` 逐渐抛弃了，因此在tools下找不到,需要自己进到android-sdk/tools/,运行monitor就可以了

获取系统版本：`adb shell getprop ro.build.version.release`
获取系统api版本：`adb shell getprop ro.build.version.sdk`
要保证程序是debuggable的

[【Android Studio】Android Monitor无法显示运行程序问题解决](https://blog.csdn.net/u012203641/article/details/73927489)  
`Can't bind to local 8601 for debugger `问题：  
看到as也可以调native层？不过没试过...   
attach 了..然后出现了Cannot find module by package name 问题 然后...一系列问题   
呜呜呜  
**动态调试示例**  
[AndroidStudio无源码动态调试apk](https://www.jianshu.com/p/1a28e6439c6a)
IDA调native:
1. `adb forward tcp:23946 tcp:23946` (使IDA连上设备 映射)
2. IDA 选择remote linux debugger `hostname:127.0.0.1`
3. IDA:attach-process 连接IDA 
4. 传IDA的android_x86_server 到模拟器   

cd /data/tmp  
java 层continue  

1 重启adb服务

adb kill-server

adb start-server
