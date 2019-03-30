---
title: 计网 | 计算机网络学习笔记
date: 2018-10-20 10:33:51
tags:
---
《计算机网络自顶向下方法》学习笔记
<!-- more -->

## 第三章 传输层
传输层：提供进程间的逻辑连接

传输层协议
### TCP 
### UDP
报文格式
+--------------+--------------+
|   源端口     |   目的端口    |
+-----------------------------+
|   长度       |   校验和      |
+--------------+--------------+
|                             |
|            数据             |
|                             |
+-----------------------------+


udp校验需要将ip伪首部、udp报头、udp数据分为16位的字，然后进行累加（如果总长度为奇数个字节，则在最后增添一个位都为0的字节 ），最后对累加的和进行按位取反即可。
Ip伪首部包括源ip地址（4字节）、目的ip地址（4字节）、协议号（两字节）、tcp包长（2字节） ，共14字节。
发送方：将所有数据相加，如果出现0则说明有错误。

计算：
反码: 这里的反码指的是对原来的数所有位二进制取反, 既: unsigned int A 的反码为~ A

对每个数求反码然后相加, 如果最高位有进位则加到最低位 (循环进位)
先取反后相加与先相加后取反，得到的结果是一样的。
如果是奇数个字节, 将最后一个字节 (设为 Z) 扩展成: [Z, 0x00]
具体可以看[RFC](https://tools.ietf.org/html/rfc1071)
使用反码的原因：不依赖系统是大端小端。，用反码求和时，交换16位数的字节顺序，得到的结果相同，只是字节顺序相应地也交换了；而如果使用原码或者补码求和，得到的结果可能就不同。
二进制反码循环移位加法与字节序无关
### rdt
rdt 1.0 :无特殊行为
rdt 2.0 差错检测
rdt 2.1 相对于rdt2.0，用序号标识分组。
rdt 2.2  a NAK-free protocol
rdt3.0: channels with errors and loss（交替位协议）
[推荐链接](http://www2.ic.uff.br/~michael/kr1999/3-transport/3_040-principles_rdt.htm)
停等协议
FSM(有限状态机)


选择性重传
TCP 快速重传
### pipelined 协议
#### Go-Back-N
receiver only sends
cumulative ack
#### Selective Repeat (SR)
Selective Repeat (SR) protocols avoid unnecessary retransmissions by having the sender retransmit only those packets that it suspects were received in error


### tcp
#### 流量控制
流量控制所要做的就是抑制发送端发送数据的速率，以便使接收端来得及接收。
#### 拥塞控制
慢开始，拥塞避免，快重传，快速恢复
拥塞窗口 **cwnd** ( congestion window )
1. 慢开始算法
慢开始门限ssthresh
当 cwnd < ssthresh 时，使用上述的慢开始算法
当 cwnd > ssthresh 时，停止使用慢开始算法而改用拥塞避免算法


两者算法大致一致，对于丢包事件判断都是以重传超时（retransmission timeout，RTO）和重复确认为条件，但是对于重复确认的处理，两者有所不同：
**Tahoe**：如果收到三次重复确认——即第四次收到相同确认号的分段确认，并且分段对应包无负载分段和无改变接收窗口——的话，Tahoe算法则进入快速重传，将慢启动阈值改为当前拥塞窗口的一半，将拥塞窗口降为1个MSS，并重新进入慢启动阶段。[15]
**Reno**：如果收到三次重复确认，Reno算法则进入快速重传，只将拥塞窗口减半来跳过慢启动阶段，将慢启动阈值设为当前新的拥塞窗口值，进入一个称为“快速恢复”的新设计阶段。

参考连接
https://www.cnblogs.com/cielosun/p/6993740.html
http://www2.ic.uff.br/~michael/kr1999/3-transport/3_040-principles_rdt.htm
