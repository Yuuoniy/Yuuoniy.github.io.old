---
title: CTF | 逆向入门第一篇
date: 2018-08-03 00:24:43
tags: xman reversing
---
跟着萌新一起逆向入门呦。包括：基本知识点,简单题例子,常见操作
<!-- more --> 
先来过一遍知识点吧！
####  工具介绍

- Disassembler 反汇编器
- Tracer 记录运行的过程,如函数，指令
- Debugger 调试器
- Decompiler IDA、geb（支持mips、web assembly?)
- Emulator (qemu 模拟执行）知道CPU状态
- Symbolic execution 符号执行 angr、路径爆炸

#####  具体工具
- IDA 支持脚本、手工定义的反汇编规则
- Geb 逆安卓 mips web***
- Binary ninja 不支持反编译,api好用，通常提供静态分析
- R2 命令系统奇怪，支持以太坊智能合约
- Retdec 
- capstone

**常规逆向**
ELF/PE/Mach-O file with x86/x64/arm  
此外还有一些非常常规逆向，比如...不说啦哈哈哈哈  
逆向的一般步骤：**信息收集->定位关键代码->分析***
####  信息收集
Strings(搜看看有没有源码..）/ binwalk file ida -> google/github  
**例子**  
- CISCN 2018 re
- 酷狗音乐盒
####  定位关键代码
- 根据控制流
- Data 交叉引用
- 代码交叉引用
- Memory searching +r/w breakpoint.输入数据  内存搜索 记录地址 对该地址下断点 读写断点（保证运行地址不变？）
- Tracing 
    **例子**：youku client reversing 

#### 分析代码 
**Tips**
- 代码是人写的 
    - 区分库代码和程序代码 （库代码也不用仔细逆）
    - 区分人写的和编译器加上的代码
    - 不需要仔细逆向编译器加上的代码
    - Binary 由编译器生成
        - Binary layout pattern 即生成的程序有规律
        - |executable|lib1|lib2|lib3|
        - 识别出编译器优化的代码
- 代码重用
    - String xref
    - Code style
- 文件是可执行的
    - 善用动态调试
        - 找到关键代码，验证猜测
        - Debugging 
        - Tracing
        - Symbolic execution
        - Taint analysis
    • 七分猜

**技巧：一块一块看而不是一行一行**
- 常见算法（xxsearch ida 插件自动识别加密算法）
    -Tea/Xtea/
- 常见数据结构
- 常见设计模式
    -Proxy stub/工厂模式
- 常见框架

#### 混淆
#####  例子：
- Ollvm (平坦的控制流 主分发器）
- ovfus。。 去掉分支
- Push rax,ret (把所有的跳转 jump rax) mov rax 0x222, 模拟执行看rax 是什么 patch 掉啥？？ 
- Vm/self modify code

#####  解混淆
    • 知道混淆的原理，恢复控制流
    • 模拟执行/符号执行
https://  https://security.tencent.com/index.php/blog/msg/112
反混淆：搜索：符号执行 ollvm




#### 壳（packer)
#####  类型：
    • 解压->运行
    • 解压->运行->解压..->运行
    • 解压 decoder|encoded code->decode ->exc
    • Run the virtual machine 
##### 解决方法
- Case by case
- Esp law (oep 找到？？)
- Read /proc/[pid]/mem

####  反调试
搜索：Windows  anti-debugging
#####  Debugger detection
- API call,isDebuggerPresent()
- Try {int3},catch{}
- Timestamp

##### Debugger Interfering
- 如果有内核权限 debugport overweite
- Self debugging 父进程调试子进程
    
    


大概是首先分析一下代码，并且使用IDA一波操作修改一下变量名、平衡堆栈、定义函数等就开始好逆。  
**IDA的一些操作**:
- 修改变量名 N
- 修改函数类型
- 修改变量类型 Y(常见有修改为数组提高可读性)
- 数字转化为char R
- Nop
- Patch 
- 取消/定义函数 U/P
- 交叉引用发现问题时，自己在函数处按P定义函数
- 修改函数结尾
- 脚本执行 (IDA portable版本中不支持python执行...)
- 修改函数结尾地址
- 查看并修改sp(options->general)
- 动态调试
- options->complier 修改编译器
- 模糊符号匹配
快捷键需要熟悉噢！
**插件**
- lazyIDA dump 数据超方便
- keypatch 
- rizoo

**参考资料**  
- https://wenku.baidu.com/view/bb468fff910ef12d2af9e7c3.html  
下面就讲几道题叭！
源程序..有空再传到github上...需要可以直接找我呀
####  re0
首先需要对judge进行动态解密,解出来后修改judge函数尾(右键->edit function) U 取消定义 P定义函数,这样就能获得正确的judge函数啦！(因为IDA是程序一开始拖进来就分析好全部的函数栈帧了，因此这里不重新定义函数的话IDA的函数栈帧没有进行更新)  
##### 动态解密
python 版
```
import ida_bytes
buf = ida_bytes.get_bytes(0x600B00, 182)
a = ""
for i in xrange(0, len(buf)):
    a += chr(0xC ^ ord(buf[i]))
ida_bytes.patch_bytes(0x600B00, a)
```
idc 版
```c
#include <idc.idc>
static main() {
    auto cur = 0x600B00;
    auto i;
    for (i = 0; i <= 0xB5; i++) {
        auto c = Byte(cur) ^ 0xC;
        PatchByte(cur, c);
        cur = cur + 1;
    }
}
```
然后进行解密：
```python
flag_enc="fmcd\x7fk7d;V\x60;np"
flag=""
for i in range(len(flag_enc)):
    c=flag_enc[i]
    flag+=chr(ord(c)^i)

print flag
```

#### RE1
首先分析程序
尝试像atum那样直接patch 程序dump 出来..始终没弄清楚
直接使用OD动态看了
首先需要过掉长度的检查，因此输入长度需要为33
之后程序就可以执行下一验证逻辑
在00x401669 处下断点(F2)
可以依次得到flag

#### simplecheck 
2018 强网杯的题
在jeb中打开，找到主代码逻辑，进行分析
首先定义a、b、c、d 四个各组
并且要求输入的字符串长度等于b 的长度，即34
接下来，创建数组V4，v4[0]=0，将输入的字符串添加到v4后面
后面是if验证 相当于一元二次方程，直接写脚本爆破即可，脚本如下：
```python
from string import printable
a = [0, 146527998, 205327308, 94243885, 138810487, 408218567, 77866117, 71548549, 563255818, 559010506, 449018203, 576200653, 307283021, 467607947, 314806739, 341420795, 341420795, 469998524, 417733494, 342206934, 392460324, 382290309, 185532945, 364788505, 210058699, 198137551, 360748557, 440064477, 319861317, 676258995, 389214123, 829768461, 534844356, 427514172, 864054312];
b = [13710, 46393, 49151, 36900, 59564, 35883, 3517, 52957, 1509, 61207, 63274, 27694, 20932, 37997, 22069, 8438, 33995, 53298, 16908, 30902, 64602, 64028, 29629, 26537, 12026, 31610, 48639, 19968, 45654, 51972, 64956, 45293, 64752, 37108];
c = [38129, 57355, 22538, 47767, 8940, 4975, 27050, 56102, 21796, 41174, 63445, 53454, 28762, 59215, 16407, 64340, 37644, 59896, 41276, 25896, 27501, 38944, 37039, 38213, 61842, 43497, 9221, 9879, 14436, 60468, 19926, 47198, 8406, 64666];
d = [0, -341994984, -370404060, -257581614, -494024809, -135267265, 54930974, -155841406, 540422378, -107286502, -128056922, 265261633, 275964257, 119059597, 202392013, 283676377, 126284124, -68971076, 261217574, 197555158, -12893337, -10293675, 93868075, 121661845, 167461231, 123220255, 221507, 258914772, 180963987, 107841171, 41609001, 276531381, 169983906, 276158562];
flag=''
for i in range(len(c)):
  for j in printable:
    if(a[i]==b[i]*ord(j)*ord(j)+c[i]*ord(j)+d[i]):
      flag+=j
flag+="}"
print flag
```

#### Evr1
对输入的flag进行加密，与已知的密文比较。分析程序看到许多反编译的以及干扰的，提取出关键部分,flag主要分为5部分，进行了4次变换，第一次变换，对于flag的每个字符。后面3次变换均为7字符
- 变换0：对flag进行变换，存到flag_tr0 中   
- 变换1：对flag_str0的7-13进行变换，存到flag_str1[0-7]     
- 变换2：对flag_str0的14-20进行变换，存到flag_str2[0-6]  
- 变换3 :对flag_str0的21-27进行变换，存到flag_str3[0-6]   
flag、flag_str0、byte_xxxx … 在地址上连续   
对flag_tr0分段赋值   
比较flag_tr0与密文，相等即可，比较位数为35   
因此使用脚本爆破即可   
```python
flag_enc=[0x1E, 0x15, 0x02, 0x10, 0x0D, 0x48, 0x48, 0x6F, 0xDD, 0xDD, 0x48, 0x64, 0x63, 0xD7, 0x2E, 0x2C, 0xFE, 0x6A, 0x6D, 0x2A, 0xF2, 0x6F, 0x9A, 0x4D, 0x8B, 0x4B, 0x0A, 0x8A, 0x4F, 0x45, 0x17, 0x46, 0x4F, 0x14,0xb];
flag=''
#变换 0
for i in range(7):
  flag+=chr(flag_enc[i]^0x76)

for i in range(7):
  for c in range(0x20,0x7f):
    originc = c
    c = c^0x76^0xAD;
    c = 2*c&0xAA|((c&0xAA)>>1)
    if(c==flag_enc[i+7]):
      flag+=chr(originc)


for i in range(7):
  for c in range(0x20,0x7f):
    originc = c
    c = c^0x76^0xBE;
    c = 4*c&0xCC|((c&0xCC)>>2)
    if(c==flag_enc[i+14]):
      flag+=chr(originc)

for i in range(7):
  for c in range(0x20,0x7f):
    originc = c
    c = c^0x76^0xEF;
    c = 16*c&0xF0|((c&0xF0)>>4)
    if(c==flag_enc[i+21]):
      flag+=chr(originc)

  
for i in range(7):
  flag+=chr(flag_enc[i+28]^0x76)

print flag
```
hctf{>>D55_CH0CK3R_B0o0M!-9193a09b}
嘤嘤嘤我实在是懒得传图  
有些地方偷了些懒，有问题可以联系我噢，应该有错的，也会随着日后学习中进行修补。
