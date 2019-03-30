---
title: CTF | reversing-kr-writeup
date: 2018-09-01 20:51:01
tags: reverse
categories: CTF
---
记录 reversing.kr 上的一些题
<!-- more -->
### easy_keygen
首先读取 name,经过处理存在res中
读取serial，比较res与serial是否相同
因此通过已知的serial反推name就是flag了
注意变换时有个%2x,即将变化转化为十六进制，因此需要decode 



### Easy_UnpackMe
简单脱壳联系
有个大jmp
找到 OEP


### easy_elf
分析
读取字，存在bss段
对于一些位进行异或变换,判断变换后的结果。因此写个简单的脚本反推就好了。
有个负数-35，应该转化为十六进制看为`0xdd`
返回值是1时正确

### music player
一开始没有思路

先打开个音频文件播放，到一分钟时有弹框，字符串“1???????"
得知要了解vb,弹框的API
下载 vb complier,代码结构看起来清晰许多
VB弹窗调用的API是rtcMsgBox 

查看API:反汇编窗口，右键--查找--当前模块中的名称，就可以列出所有函数了。快捷键Ctrl+N 
右键查看引用

找到弹框，找到关键跳转，修改为jmp
抛出异常 380 


对不起..我应该插图的

