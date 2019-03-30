---
title: 算法 | 刷题记录 
date: 2018-11-02 10:13:37
tags:
---
牛客网
计算机考研/保研机试
<!-- more -->

## 递推数列
这道题如果先计算出a[k]的值再取模，会造成溢出，因此要边计算边取模
```c
#include <iostream>

using namespace std;

int main(){
  int p,q,k;
  long long a[1000];
  cin >> a[0] >> a[1] >> p >> q >> k;
  for(int i =2;i<=k;i++){
    a[i] = p * a[i - 1] % 10000 + q * a[i - 2] % 10000;
  }
  cout << a[k]%10000 << endl;

}
```
