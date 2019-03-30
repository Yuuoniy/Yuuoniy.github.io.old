---
title: 信安 | DES 原理讲解及实现
date: 2018-10-19 21:01:21
tags:
---
DES 加密机制讲解及 c++实现
<!-- more -->


[toc]

<!-- /code_chunk_output -->
## 算法原理概述
- `DES(Data Encryption Standard)` 是一种典型的块密码，一种将固定长度的明文通过一系列复杂操作编程同样长度的密文的算法。
- `DES` 算法的基本过程是换位和置换。对于`DES`而言，块长度为`64`位。
- 同时，`DES`使用密钥来自定义变换过程。因此算法认为只有持有加密所用的密钥的用户擦能解密密文。
- 密钥表面上是`64`位的，但是只有其中的56位被实际用于算法，其余8位可以被用于奇偶校验。密钥可以是任意的56位的数，且可随时改变。其中极少量的数被认为是弱密钥，但能容易地避开它们。所有的保密性依赖于密钥。


## 总体结构
DES 原始明文消息按 **`PKCS#5 (RFC 8018)`** 规范进行字节填充。
- 原始明文消息最后的分组不够8个字节 (64位) 时，在末尾以
字节填满，填入的字节取值相同，都是填充的字节数目；
- 原始明文消息刚好分组完全时，在末尾填充8个字节 (即增
加一个完整分组)，字节取值都是08。
明文分组结构：$M = m_1m_1 … m_64 , m_i\in{0, 1}, i = 1 .. 64.$
密文分组结构：$C = c_1c_2 … c_64 , c_i \in{0, 1}, i = 1 .. 64$
密钥结构：$K = k_1k_2 … k_64 , k_i \in{0, 1}, i = 1 .. 64.$
  除去 $k_8, k_16, …, k_64$ 共8位奇偶校验位，有效密钥为56位

算法的总体结构如图所示：
![enter image description here](./pics/des.png)
密钥：$\{K_1, K_2, …, K_{16}\}$
### 加密过程
1. **IP 置换**
2. **16 轮 T 迭代**：其中包括主要函数：**F 轮函数**，而 F 轮函数中的 **S 盒**非线性变换保证了 DES 的安全性。加密与解密密钥调度的不同也在T迭代时体现。
3. **W 置换** 
4. $IP^{-1}$ 的逆置换

使用数学公式表示如下：
$C = E_k(M) = IP^{-1} · W · T_{16} · T_{15} ·… · T_1 · IP(M).$
- M 为算法输入的 64 位明文块；
- $E_k$ 描述以 K 为密钥的加密函数，由连续的过程复合构成；
- IP 为64位初始置换
- $T_1, T_2 , …, T_{16}$ 是一系列的迭代变换
- W 为64位置换，将输入的高32位和低32位交换后输出；
- $IP^{-1}$ 是 `IP `的逆置换；
- C 为算法输出的64位密文块

### 解密过程
解密和加密相似，只是密钥调用不一样
$M = D_k(C) = IP^{-1} · W · T_1 · T_2 · … · T_{16} · IP (C)$


## 模块分解
### 初始置换
64 位明文串经过一个初始置换函数`IP`，生成重新排列之后的64位输出，分为左32位$L_0$和右32位$R_0$
IP 置换表：
```py
IP 置换表 (64位)
58 50 42 34 26 18 10 2
60 52 44 36 28 20 12 4
62 54 46 38 30 22 14 6
64 56 48 40 32 24 16 8
57 49 41 33 25 17 9 1
59 51 43 35 27 19 11 3
61 53 45 37 29 21 13 5
63 55 47 39 31 23 15 7
```
### 迭代T
根据 $L_0R_0$ 按下述规则进行16次迭代，即
$L_i = R_{i-1}, R_i = L_{i-1} \bigoplus f(R_{i-1,} K_i), i = 1 .. 16.$
- 16 个长度为 48 位的子密钥 $K_i (i = 1 .. 16)$ 由密钥 K 生成；
- 16 次迭代后得到 $L_{16}R_{16}$；
- 左右交换输出 $R_{16}L_{16}$。
- f 为 `Feistel` 轮函数，在后面进行解释


### 逆置换 $IP^{-1}$
对迭代 T 输出的二进制串 $R_{16}L_{16}$ 使用初始置换的逆置换 $IP^{-1}$ 得到
密文 C，即：$C = IP^{-1} (R_{16}L_{16})$.
```python
IP-1 置换表 (64位)
40 8 48 16 56 24 64 32
39 7 47 15 55 23 63 31
38 6 46 14 54 22 62 30
37 5 45 13 53 21 61 29
36 4 44 12 52 20 60 28
35 3 43 11 51 19 59 27
34 2 42 10 50 18 58 26
33 1 41 9 49 17 57 25
```
### Feistel 轮函数 $f(R_{i-1}, K_i)$
轮函数主要步骤：**E 拓展->S 盒置换->P 置换**
详细说明：
1. 将长度为 32 位的串 $R_{i-1}$ 作 **E-扩展**，成为48位的串 $E(R_{i-1})$；
2.  将 $E(R_{i-1})$ 和长度为48位的子密钥 $K_i$ 作 48 位二进制串按位异或运
算，$K_i$ 由密钥 K 生成；
3. 将 2 得到的结果平均分成 8 个分组，每个分组长度 6 位。各个
分组分别经过 8 个不同的 **S-盒**进行 `6-4` 转换，得到8个长度分别
为4位的分组；
4. 将 3 得到的分组结果顺序连接得到长度为`32`位的串
5. 将 4 的结果经过 **P-置换**，得到的结果作为轮函数 $f(R_{i-1}, K_i
)$ 的最终32位输出。
#### E 拓展
作用是将`32`为扩展为`48`位，其扩展规则按下表进行。该表大小为`6*8`，去掉最左和最右两行，中间的四列包含了`1-32`的顺序。第1列和第6列分别是对第4和第1列的扩展。经过扩展置换，`32`位的输入得到了48位的输出。
```py
E-扩展规则 (比特-选择表)
32 1 2 3 4 5
4 5 6 7 8 9
8 9 10 11 12 13
12 13 14 15 16 17
16 17 18 19 20 21
20 21 22 23 24 25
24 25 26 27 28 29
28 29 30 31 32 1
```
#### S 盒
- S-盒是一类选择函数，用于二进制 `6-4` 转换。 `Feistel` 轮函数
使用8个 S-盒 $S_1, …, S_8$，每个 S-盒是一个 4 行 (编号 0-3)、16
列 (编号 `0-15`) 的表，表中的每个元素是一个十进制数，取
值在`0-15`之间，用于表示一个4位二进制数。
- S盒运算其实是一个查表运算。在E盒的扩展之后得到了48位的数据，将其和48位的子密钥进行异或运算，这是密钥参与运算的步骤。将其分成8个组，每组6个，送到8个S盒中去。
- 假设 $S_i$ 的6位输入为 $b_1b_2b_3b_4b_5b_6$，则由 $n = (b_1b_6)_{10}$ 确定行
号，由 $m = (b_2b_3b_4b_5)_{10}$ 确定列号，$[S_i]_{n,m}$ 元素的值的二进制
形式即为所要的 $S_i$ 的输出。
-  S 盒的作用是进行非线性代换。S盒是`DES`中唯一的非线性部分，`DES`的安全强度主要取决于S盒的安全程度。

```py
S1-BOX
14 4 13 1 2 15 11 8 3 10 6 12 5 9 0 7
0 15 7 4 15 2 13 1 10 6 12 11 9 5 3 8
4 1 14 8 13 6 2 11 15 12 9 7 3 10 5 0
15 12 8 2 4 9 1 7 5 11 3 14 10 0 6 13
S2-BOX
15 1 8 14 6 11 3 4 9 7 2 13 12 0 5 10
3 13 4 7 15 2 8 14 12 0 1 10 6 9 11 5
0 14 7 11 10 4 13 1 5 8 12 6 9 3 2 15
13 8 10 1 3 15 4 2 11 6 7 12 0 5 14 9
S3-BOX
10 0 9 14 6 3 15 5 1 13 12 7 11 4 2 8
13 7 0 9 3 4 6 10 2 8 5 14 12 11 15 1
13 6 4 9 8 15 3 0 11 1 2 12 5 10 14 7
1 10 13 0 6 9 8 7 4 15 14 3 11 5 2 12
S4-BOX
7 13 14 3 0 6 9 10 1 2 8 5 11 12 4 15
12 8 11 5 6 15 0 3 4 7 2 12 1 10 14 9
10 6 9 0 12 11 7 13 15 1 3 14 5 2 8 4
3 15 0 6 10 1 13 8 9 4 5 11 12 7 2 14
S5-BOX
2 12 4 1 7 10 11 6 8 5 3 15 13 0 14 9
14 11 2 12 4 7 13 1 5 0 15 10 3 9 8 6
4 2 1 11 10 13 7 8 15 9 12 5 6 3 0 14
11 8 12 7 1 14 2 13 6 15 0 9 10 4 5 3
S6-BOX
12 1 10 15 9 2 6 8 0 13 3 4 14 7 5 11
10 15 4 2 7 12 9 5 6 1 13 14 0 11 3 8
9 14 15 5 2 8 12 3 7 0 4 10 1 13 11 6
4 3 2 12 9 5 15 10 11 14 1 7 6 0 8 13
S7-BOX
4 11 2 14 15 0 8 13 3 12 9 7 5 10 6 1
13 0 11 7 4 9 1 10 14 3 5 12 2 15 8 6
1 4 11 13 12 3 7 14 10 15 6 8 0 5 9 2
6 11 13 8 1 4 10 7 9 5 0 15 14 2 3 12
S8-BOX
13 2 8 4 6 15 11 1 10 9 3 14 5 0 12 7
1 15 13 8 10 3 7 4 12 5 6 11 0 14 9 2
7 11 4 1 9 12 14 2 0 6 10 13 15 3 5 8
2 1 14 7 4 10 8 13 15 12 9 0 3 5 6 11
```
#### P 置换
进行简单的位置置换，只是简单地把一位换成另一位，不进行扩展和压缩。经过P盒操作，`32`位的输入得到`32`位的输出。
```py
P-置换表
16 7 20 21
29 12 28 17
1 15 23 26
5 18 31 10
2 8 24 14
32 27 3 9
19 13 30 6
22 11 4 25
```
### 子密钥生成
子密钥生成过程根据给定的`64`位密钥 K，生成`16`个`48`位的子密
钥 $K_1-K_{16}$，供 `Feistel` 轮函数 $f(R_{i-1}, K_i)$ 调用。
1. 对 K 的`56`个非校验位实行置换 **PC-1`**，得到 $C_0D_0$，其中 $C_0 和 D_0$
分别由 `PC-1` 置换后的前 28 位和后 28 位组成。
2. 计算 $C_i = LS_i(C_{i-1})$ 和 $D_i = LS_i(D_{i-1})$
  -  当 $i =1, 2, 9, 16$ 时，$LS_i(A)$ 表示将二进制串 A 循环左移一个
位置；否则循环左移两个位置。
3. 对 `56`位的 $C_iD_i$ 实行 **PC-2 压缩置换**，得到48位的 $K_i$ 。i = i+1。
4.  如果已经得到 $K_{16}$，密钥调度过程结束；否则转 2。
**PC-2 压缩置换**：从56位的 $C_iD_i$ 中去掉第 $9, 18, 22, 25, 35, 38, 43,54$位，将剩下的48位按照 `PC-2` 置换表作置换，得到 $K_i$ 。
子密钥生成流程图:
![enter image description here](./pics/subkey.png)
```py
PC-1 置换表
57 49 41 33 25 17 9
1 58 50 42 34 26 18
10 2 59 51 43 35 27
19 11 3 60 52 44 36
63 55 47 39 31 23 15
7 62 54 46 38 30 22
14 6 61 53 45 37 29
21 13 5 28 20 12 4
```
### 密钥调度
DES 加解密过程中使用由同一个密钥 `K`经过相同的子密钥生
成算法得到的子密钥序列，唯一不同之处是加解密过程中子密
钥的调度次序恰好**相反**。
- 加密过程的子密钥按 $(K_1 K_2 … K_{15} K_{16}) $次序调度
- 解密过程的子密钥按 $(K_{16} K_{15} … K_2 K_1) $次序调度
## 数据结构
加解密主要用到的结构：
1. $IP$ 和 $IP^{-1}$置换表
2. 轮函数F
  - E盒 扩展置换
  - S盒 非线性代换
  - P盒 线性置换
3. 子密钥生成
  - PC-1
  - PC-2 

实现过程中涉及到的数据结构：
一维数组存储：$IP$,$IP^{-1}$ E 盒 S 盒 P 盒
三维数组存储：S 盒
`vector` 存储：传入的数据
`bitset` 存储：加密过程中位数据，比如子密钥

## C 语言源代码
实现一个 des 类,完整代码` des.h、 des.cpp `和 `main.cpp` 中，此处贴出关键部分。
des 包含的成员变量及成员方法：
```c++
class des{
  private:
  vector<unsigned char> data; //传入 des 的数据
  vector<unsigned char> result; //经过des 运算得到的结果
  vector<unsigned char> inputKey; //输入的key 
  bitset<64> key; // inputKey 的位表示
  int mode; // 标记是加密还是解密，1 是加密，0是解密
  int len; //数据的长度，单位是字节
  const static unsigned char PC1[];
  const static unsigned char PC2[];
  const static unsigned char IP[];
  const static unsigned char IpInverse[];
  const static unsigned char sBox[8][4][16];
  const static unsigned char eBox[];
  const static unsigned char pBox[];
  bitset<48> subKeys[16]; //生成的子密钥

public:
  des();
  vector<unsigned char> desProcess(vector<unsigned char> data, vector<unsigned char> key, int mode);
  bitset<64> TIteration(bitset<64> &dataBit); //T 迭代
  void subKeyGenerated();    //子密钥生成
  bitset<32> sBoxTransform(bitset<48> &R); //s 盒变换
  void PKCS5Padding();       //字节填充
  bitset<32> feistel(bitset<32> R, bitset<48> key); // F 轮函数
  void leftShift(); //左移
  bitset<64> encrypt(bitset<64> &plain); // 加密
  void decrypt(); // 解密
  void transform(bitset<64>& data, int originSize, const int resSize, const unsigned char *table);
  void charToBits(vector<unsigned char> &str, bitset<64> &bits);
  void bitsToChar(bitset<64> &bits, vector<unsigned char> &str);
```
### 关键函数
####  des 主过程 `desProcess`
接收三个参数：
1. `data`:需要操作的数据
2. `key` 为`8`字节的密钥
3. `mode` 表示模式，1 代表加密，0 代表解密。
首先判断 `key` 的长度是否合法。如果是加密需要对数据进行填充，填充函数为 `PKCS5Padding`。若是解密则需要最后删除填充的字节。
```c++
vector<unsigned char> des::desProcess(vector<unsigned char> _data, vector<unsigned char> _key, int _mode){
  this->data = _data;
  this->inputKey = _key;
  this->mode = _mode;
  //判断 key 长度是否合法
  if (_key.size() != 8){
    cout << "Invalid key" << endl;
    return result;
  }
  if (mode == 1){ //encrypt
    len = data.size() + 8 - data.size() % 8;
    PKCS5Padding();
  }
  else
  {
    len = data.size();
  }
  result.resize(len);
  // 生成16个子密钥
  charToBits(inputKey, this->key);
  subKeyGenerated(); //生成子密钥
  //padding
  int times = this->data.size() / 8;
  //分块进行那个加密，每块8字节
  for (int i = 0; i < times; i++)
  {
    vector<unsigned char> rawData(8);
    for (int j = 0; j < 8; j++)
      rawData[j] = data[i * 8 + j];
    bitset<64> dataBit,processBit,cipher;
    charToBits(rawData, dataBit); //转为64 位
    //IP 变换
    for (int i = 0; i < 64; ++i)
      processBit[63 - i] = dataBit[64 - IP[i]];
    //T 迭代 和 W 置换
    TIteration(processBit);
    // IP 逆置换
    cipher = processBit;
    for (int i = 0; i < 64; ++i)
      cipher[63 - i] = processBit[64 - IpInverse[i]];

    //从位转化为字节
    bitsToChar(cipher, rawData);
    for (int j = 0; j < 8; j++)
      result[i * 8 + j] = rawData[j];
  }
  //填充删除PKCS5填充
  if (mode == 0)
  {
    result[len - result[len - 1]] = '\0';
  }
  cout << "reslut is:" << endl;

  return result;
}
```
#### 生成子密钥：`subKeyGenerated`
```c
void des::subKeyGenerated(){
  bitset<56> realKey;
  // PC1 置换
  for(int i =0;i<56;i++)
    realKey[55-i] = key[64]-PC1[i];
  //生成子密钥
  for (int i = 0; i < 16; i++)
  {
    if (i != 0 && i != 1 && i != 8 && i != 15)
      leftShift(); //左移
    leftShift();
    bitset<64> keyCopy;
    for(int i=0;i<64;i++)
      keyCopy[i] = key[i];
    // PC2 变换
    for(int i=0;i<48;i++){
      key[47 - i] = keyCopy[56 - PC2[i]];
    }
    for (int j = 1; j <= 48; j++)
      subKeys[i][j] = key[j];
    key = keyCopy;
  }
}
```
#### 左移函数：`leftShift `
分前后两部分进行左移
```c
void des::leftShift(){
 unsigned char chead = key[0],dhead = key[28];
 for(int i = 0; i < 27; i++)
 {
   key[i] = key[i+1];
   key[i+28] = key[i+28+1];
 }
 key[27] = chead;
 key[55] = dhead;
}
```
#### T 迭代+W置换 `TIteration`
对数据进行`16`轮变换，最后合并，注意这里需要判断是解密还是加密，对密钥进行调度
```c
bitset<64> des::TIteration(bitset<64>&dataBit){
  bitset<32> left;
  bitset<32> right;
  bitset<32> newLeft;
  bitset<64> reslut;
  for (int i = 32; i < 64; ++i)
    left[i - 32] = dataBit[i];
  for (int i = 0; i < 32; ++i)
    right[i] = dataBit[i];
  for(int i = 0; i < 16; i++)
  {
    newLeft = right;
    if(mode==1)//加密
      right = left ^ feistel(right, subKeys[i]);
    else if(mode==0) //解密
      right = left^feistel(right,subKeys[15-i]);
    left = newLeft;
  }
  // 合并L16 和 R16
  for(int i = 0; i < 32; i++)
  {
    reslut[i]=left[i];
    reslut[i+32] = right[i];
  }
  dataBit = reslut;
}
```
#### F 轮函数 `feistel`
```c++
bitset<32> des::feistel(bitset<32> R, bitset<48> key)
{
  bitset<48> expandR;
  //E 拓展
  for(int i=0;i<48;i++){
    expandR[47-i] = R[32-eBox[i]];
  }
 //异或
  expandR = expandR^key;
 // s 盒变换返回32位的串
  bitset<32> res = sBoxTransform(expandR);
  bitset<32> tmp = res;
	  // p 置换
  for(int i=0;i<32;i++)
    res[31-i] = tmp[32-pBox[i]];
  return res;
}
```
#### S 盒变换 `sBoxTransform`
```c++
bitset<32> des::sBoxTransform(bitset<48> &src)
{
  bitset<32> result;
  for(int i =0;i<8;i++){
    int begin = 6*i,row = src[begin]*5+src[begin+5],
    col = src[begin+1]*8+src[begin+2]*4+src[begin+3]*2+src[begin+4];
   int  s = sBox[i][row][col];
    //转化为二进制
    bitset<4> bin(s);
    for(int j=0;j<4;j++){
      result[i*4+4-j] = bin[j];
    }
  }
  return result;
}
```
测试代码为 `main.cpp `
因为`DES`为对称加密，因此可以使用`DES`加密后的密文再调用`DES`进行解密，判断解密出的明文与源字串是否一致即可。
这里我加密的数据是 `"helloworld"` 密钥是`"sysucsas"`
```c++
des desOject;
vector<unsigned char>cipher =  desOject.desProcess(data, key, 1);
cout << "encrypt result:";
for (int i = 0; cipher[i] != '\0' && i < cipher.size(); i++)
  cout << cipher[i];
cout << endl;

vector<unsigned char>plain =  desOject.desProcess(cipher, key, 0);

cout  <<"decrypt result:";
for (int i = 0; plain[i] != '\0' && i < plain.size(); i++)
  cout << plain[i];
cout << endl;
```

## 编译运行结果
编译运行：`g++ des.cpp main.cpp -g -o des`
可以看到，加密 `helloworld` 得到密文，因为有不可打印字符，所以显示有问题。拿该串密文进行解密，成功得到原始明文，运算结果正确。

![enter image description here](./pics/des-res.png)

## 总结
1. 学习 `DES` 的原理结构时，首先需要从全局把握主要的过程：`IP` 置换、`T` 迭代、`W` 置换、$IP^{-1}$置换。之后对每个步骤进行理解，尤其是 T 迭代部分的 F 轮函数，然后对于 F 轮函数中的 E盒，S盒，P盒进行学习理解。其中最主要的便是 S 盒。除此之外，还有一些细节方面的操作，比如数据填充，左移操作等。首先从全局出发，再对各个模块不断深入学习，有利于 `DES` 的学习掌握。
在实现 `DES` 时，首先需要确定变量以及模块的函数。`DES` 中很多地方都有置换，置换
2. 我认为实现 `DES` 时逻辑上不是很复杂，只需要把各个步骤适当地分模块，DES 中的置换，移位，异或等操作实现起来都是比较简单的。但是需要注意很多细节的部分，比如数组下标、左右两部分、数据填充等等。另外还需要适当选取存储方式，比如 `vector` 还是指针数组、bitset 还是 char 。
3. 通过本次对 DES 原理的学习，以及动手实现。对于 DES 有了深入的理解和认识。以前认为实现 DES 是很复杂的一件事，一步一步分模块实现后感觉没有自己想的那么难，也很有成就感。虽然过程中也会遇到一些坑，我没想到 `vscode` 自带的终端输出竟然有问题。一直以为自己代码写错了，之后发现是 vscode 终端的问题，有些数据没有正常输出。换了终端就好了。
## 参考资料 
[DES 维基百科](https://zh.wikipedia.org/wiki/%E8%B3%87%E6%96%99%E5%8A%A0%E5%AF%86%E6%A8%99%E6%BA%96)
信息安全课程 ppt
