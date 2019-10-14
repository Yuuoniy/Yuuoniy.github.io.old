---
title: 工具 | wireshark 基本知识
date: 2018-10-03 18:41:47
tags:
---
记录使用 wireshark 时的一些点
<!--  -->

## 过滤规则
ip.src eq 192.168.1.107 or ip.dst eq 192.168.1.107

过滤端口：
tcp.port eq 80 
tcp.port eq 80 // 不管端口是来源的还是目标的都显示
tcp.port == 80
tcp.port eq 2722
tcp.port eq 80 or udp.port eq 80
tcp.dstport == 80 // 只显tcp协议的目标端口80
tcp.srcport == 80 // 只显tcp协议的来源端口80


4.过滤MAC



太以网头过滤
eth.dst == A0:00:00:04:C5:84 // 过滤目标mac
eth.src eq A0:00:00:04:C5:84 // 过滤来源mac
eth.dst==A0:00:00:04:C5:84
eth.dst==A0-00-00-04-C5-84
eth.addr eq A0:00:00:04:C5:84 // 过滤来源MAC和目标MAC都等于A0:00:00:04:C5:84的


less than 小于 < lt 

小于等于 le


等于 eq

大于 gt
大于等于 ge
不等 ne


---------------------
7.TCP参数过滤
tcp.flags 显示包含TCP标志的封包。
tcp.flags.syn == 0x02     显示包含TCP SYN标志的封包。
tcp.window_size == 0 && tcp.flags.reset != 1
