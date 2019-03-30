---
title: 算法 | 快速幂取模&冒泡排序
date: 2018-11-01 19:06:27
tags: 算法
---
算法学习
<!-- more -->
## 快速幂取模
快速幂取模算法是在o( logn )的时间内求得 a ^ b % n的值
结论：`a*b % c = ( ( a % c ) * ( b % c ) ) % c`

### 算法核心
递归:
$$b \& 1 == 1: a ^ b = ( a^{b/2} ) ^ 2 * a;$$
$$b \& 1 == 0: a ^ b = ( a ^ { b / 2 } ) ^ 2;$$

递归：
```c
int pow_mod( int a, int b, int n )
{
    int t = 1;
    if( b == 0 ) return 1;
    if( b == 1 ) return a % n ;
    t = pow_mod( a, b >> 1, n );
    t = t * t % n;
    if( b & 1 ) t = t * a % n;
    return t;
}
```
非递归

```c
int pow_mod( int a, int b, int n )
{
   int t = 1, tmp = a % n;
   while( b )
   {
       if( b & 1 ) t = t * tmp % n;
       tmp = tmp * tmp % n;
       b >> = 1;
   }
   return t;
}
```

例题：
求`root(N, k)`
对于`root(N, k)`中的N，我们可以把N看作关于k的多项式，也就是$N = a0 + a1*k + a2*k^2 + … + an* k^n$，而我们要求的root函数就是这个多项式的系数和，也就是$a0 + a1 + a2 + ... + an$。

考虑$root(N^2, k)$。此时$N^2 = (a0 + a1*k + a2*k^2 + … + an* k^n)^2$，而这个多项式展开后的系数和是$(a0 + a1 + a2 + … + an)^2$
因此根据快速幂取模得到下面公式：

$y \& 1 == 0: root(x, y, k) = root((root(x, y / 2, k))^2, 1, k)$

$ y \& 1 == 1:     root(x, y, k) = root((root(x, y / 2, k))^2 * root(x, 1, k), 1, k)$ 

$root(x, 1, k) = x % k + x / k % k + ...$(大于k的话再重复求root)

代码：
```c
#include <iostream>
 
using namespace std;
 
int root(int x, int y, int k) {
    if (y == 1) {
        if (x < k)
            return x;
        else {
            while (x >= k) {
                int sum = 0;
                while (x > 0) {
                    sum += x % k;
                    x /= k;
                }
                x = sum;
            }
            return x;
        }
    }
    else {
        int result = root(x, y / 2, k);
        result *= result;
        if (y % 2 == 1)
            result *= root(x, 1, k);
        return root(result, 1, k);
    }
}
 
int main() {
    int x, y, k;
    while (cin >> x >> y >> k) {
        cout << root(x, y, k) << endl;
    }
    return 0;
}
```
## 模运算性质
$$(a + b) % p = (a % p + b % p) % p$$
$(a - b) % p = (a % p - b % p) % p$
$(a * b) % p = (a % p * b % p)$
$a ^ b % p = ((a % p)^b) % p$

## 冒泡排序
注意外层循环 i 是从 1 开始，不是0！否则 j 到达 count-1 截至，而会访问 j+1,造成数字越界。
```c
 for (int i = 1; i < count; i++)
{
  for (int j = 0; j < count - i; j++)
  {
    if (stus[j].score > stus[j + 1].score)
    {
      tmp = stus[j];
      stus[j] = stus[j + 1];
      stus[j + 1] = tmp;
    }
  }
}
``

