---
title: 算法 | openJudge 刷题记录
date: 2018-11-07 10:16:20
tags:
---
刷题记录
<!-- more -->
## 4120 硬币
### 描述
宇航员Bob有一天来到火星上，他有收集硬币的习惯。于是他将火星上所有面值的硬币都收集起来了，一共有n种，每种只有一个：面值分别为a1,a2… an。 Bob在机场看到了一个特别喜欢的礼物，想买来送给朋友Alice，这个礼物的价格是X元。Bob很想知道为了买这个礼物他的哪些硬币是必须被使用的，即Bob必须放弃收集好的哪些硬币种类。飞机场不提供找零，只接受恰好X元。

### 输入
第一行包含两个正整数n和x。（1 <= n <= 200, 1 <= x <= 10000)
第二行从小到大为n个正整数a1, a2, a3 … an （1 <= ai <= x)
#### 输出
第一行是一个整数，即有多少种硬币是必须被使用的。
第二行是这些必须使用的硬币的面值（从小到大排列）。
#### 样例输入
5 18
1 2 3 5 10
#### 样例输出
2
5 10
#### 提示
输入数据将保证给定面值的硬币中至少有一种组合能恰好能够支付X元。
如果不存在必须被使用的硬币，则第一行输出0，第二行输出空行。

#### 解题思路

看到这道题就想到找硬币问题，不过那道题是要求算出最小的硬币数，而这一道题是找到必须使用的硬币的面值。而且每个硬币只有一个。
遍历每一种硬币，如果必须被使用，那么不含有这个硬币的方法数为0；
应该也可以用动态规划
一下思路是参考别人的博客知道的：
如何求出所有硬币组成总价值为i的方法总数：
创建数组，下标为总价值，值表示形成这个价值的方法总数，
对于每一个输入的硬币，都有两种选择：
1. 不选：则kinds[j] 不变
2. 选，则 kinds[j] 变为 kinds[j-value] 的值，value 是硬币的价值

#### 代码
总体来自@BeiYu-oi
```c++
#include <iostream>
using namespace std;
int kinds[10010];
int calc(int totalMoney,int value){
  if(totalMoney<0)
    return 0;
  else
  //总方法数减去含有这一枚面值为nowvalue的硬币的方法数目;
  //即减去总价值为allvalue - nowvalue,不含这枚硬币的方法数目;
    return kinds[totalMoney]-calc(totalMoney-value,value);
}
int main(){
  int values[210],coinCount = 0;
  int totalMoney;
  int finalCoin[210]; //记录最后的硬币种类
  int cntCoin = 0;
  cin >> coinCount >> totalMoney;
  
  kinds[0] = 1;//初始条件
  for(int i = 0; i < coinCount; i++)
  {
    cin >> values[i];
    for(int j = totalMoney; j>=values[i]; --j)
    {
      kinds[j]+=kinds[j-values[i]];
    }
  }
  //若总价值为x且不含这枚硬币的方法数目为0，则必须含有这枚硬币*/
  for(int i = 0; i < coinCount; i++)
  {
    if(calc(totalMoney,values[i])==0) //代表必须包含这种硬币 
      finalCoin[cntCoin++] = values[i];
  }
  cout << cntCoin  << endl;
  if(!cntCoin)
    cout << endl;
  else{
    for(int i = 0; i < cntCoin; i++)
    {
      cout << finalCoin[i] << " ";
    }
  }
  return 0;
}
```
## 
