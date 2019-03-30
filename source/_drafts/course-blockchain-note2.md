---
title: 区块链 | 智能合约讲解
date: 2018-10-15 14:24:48
tags:
---
智能合约
<!-- more -->

## 智能合约及平台简介
新建账户
先挖矿才有钱部署合约

开启节点的RPC接口
智能合约连接
账户解锁：
合约公有变量
特点
- 去中心化执行
- 不可篡改
- 可回溯

stateRoot:
- 方便相互验证
GasPrice：汇率
Gasprice*gasused = 手续内
交易的nonce值：确认交易的顺序

智能合约的原子性！
### 以太坊
### Solidity 
语言：
- Solidity
- Serpent,Vyper
- Mutan
- LLL

Solidity:
特殊类型：address,event,以太币单位
特殊关键字：payable,send,now
公有链环境中需要确定变量位置进行付费。

在线编译器：
http://remix.ethereum.org/
https://editor.hyperchain.cn/
中文文档：
https://solidity-cn.readthedocs.io/zh/develop/

- 内部函数
- 外部函数

public/private/external/internal
internal:在当前Solidity源码中均可用
- pure:不允许修改或访问状态
- view 不允许修改状态
- payable: 允许从调用者接收以太币
- constant: 修饰状态变量时，不允许赋值
- 修饰函数时，与 view 等价

mapping:
- 映射可以视作哈希表
- 映射与哈希表不通过：映射存储key的keccak256 哈希值
- 映射没有长度
mapping(address=>unit)

特殊变量
- 
特殊函数
- require
- revert
- assert
event 事件
输出中间结果
- event EVENT()

### 联盟链智能合约
- 以太坊联盟链 PoA,parity
- Hyperledger Fabric 联盟链（chaincode)
- 
### 图形交互

强调选题和链接

