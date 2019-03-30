---
title: 多媒体技术-图像处理
date: 2018-10-06 15:28:52
tags:
---
算法，编程库介绍
<!-- more -->
python 图像处理，常用的库有 `numpy、scipy、scikit-imge、opencv `这些。
### opencv
首先安装，环境是 `ubuntu 16.04`
`sudo pip install opencv-python`
顺便装一下 `sudo pip install jupyter`

本次主要使用的是 `numpy+cv2+ matplotlib`

### 图像基本操作
```python
import cv2
import numpy
```
读取图像 
`img = cv2.imread(path)`
显示图像
`plt.imshow(img)`
或者
```python
cv2.imshow('image',img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```
复制图片
`empytimg = img.copy()`
保存图片
`cv2.imwrite(path,img)`

获取并修改像素值
```python
px=img[100,100]
print(px)
blue = img[100,100,0]
print(blue)
img[101,101]=[255,255,255]
print(img[101,101])
```
更好的方法是使用 `numpy` 中的 `array.item(),array.itemset()` 效率会更高，比如
```python
print(img.item(10,10,2))
img.itemset((10,10,2),100)
print(img.item(10,10,2))
```
获取图像属性
`img.shape` 得到 行数、列数、通道数的元组。如果是灰度，那么返回值仅有行数和列数。
`img.size` 返回图像的像素数目
`img.dtype` 返回数据类型
对图像特定区域进行操作
```python
ball =img[20:30,30:30]
img[40:40,50:50]=ball
```
拆分及合并图像通道
```python
r,g,b=cv2.split(img)#拆分
img=cv2.merge(r,g,b)#合并
```

也可以这样，会比split 快
```python
b=img[:,:,0]#拆分b通道
```
假如想使所有红色通道值都为0，不必拆分再赋值，可以使用`numpy`索引，这样更快
`img[:,:,2]=0`

我们需要实现两张图片的场景切换，第二张图片从第一张图片的中间以圆形的方式慢慢覆盖。最后的效果用 gif 展示

每张图片都是一个矩阵，我们需要对像素进行操作，那么怎么进行呢...
我们可以根据光圈大小的不同生成多个图片，再整合成 gif，这里使用 `imageio` 库
主要函数 `imageio.mimsave(exportname, frames, 'GIF', **kargs)`
具体合成操作可以看这篇博客 [【python图像处理】gif动态图的解析与合成](https://blog.csdn.net/guduruyu/article/details/77540445)

好了，下一步想想这些帧要怎么生成

其实要做的就是 利用ROI实现图像叠加，然后 ROI 是圆形的
5555 好难 让我再好好看一下....

我们自己用矩阵就可以呀! 自己实现圆形区域的选区，然后把值赋为要叠加的图片对应处的值就好了，
首先获取图像的中心点就好了！是 [204,204] 好耶，easy 我怎么这么傻，老是想着怎么用opencv中的东西，其实自己实现就好了
美妙

有一个问题，matplotlib 绘制的图像偏蓝，搜下看看为什么

好了，接下来看第二个问题，主要是颜色查找表和中值区分算法 看书看书
LUT : look up table
有点懵
opencv 中提供了LUT 函数，不想那么多，直接根据书上的自己实现好了
看了一遍，有点不清楚啊
首先建立颜色查找表，有 256 个行，每行代表 24 位的颜色。相当于我们把24位色彩空间切割成 256 小块，每个小块有一个代表颜色。当然这个颜色肯定在小块内，对于任意一张图片，我们用块号代替每一个像素值。相当于把24位RGB值转化成 8 位的块号值，而块号代表了原本的RGB值所在的区域，但是该区域全部用代表色代替了。所以图片不能无损转化。

我怎么这么笨，写个作业都要这么久

用到的内容
可以看一下 [numpy数据类型](https://wizardforcel.gitbooks.io/ts-numpy-tut/content/3.html)

不是三维数组啊 ，是 256*3 的矩阵，好傻逼。。
创建查找表
np.zeros((256,3),dtype = np.uint8)
怎么找出最小的方型区呢

分离通道
```python
b = np.zeros((img.shape[0],img.shape[1]), dtype=img.dtype)
g = np.zeros((img.shape[0],img.shape[1]), dtype=img.dtype)
r = np.zeros((img.shape[0],img.shape[1]), dtype=img.dtype)

b[:,:] = img[:,:,0]
g[:,:] = img[:,:,1]
r[:,:] = img[:,:,2]
 
cv2.imshow("Blue",b)
cv2.imshow("Red",r)
cv2.imshow("Green",g)

```
很坑注意顺序是 bgr 
 
感觉 这个教程很多错误 https://www.kancloud.cn/aollo/aolloopencv/262768 ,其他博客也很多错误，真的很坑。
还是好好看官方文档吧。
是 b,g,r = cv2.split(img) 而不是  r,g,b=cv2.split(img)!!

Python中可以通过以"0b"或者"-0b"开头的字符串来表示二进制 
这里 我稍微用了bitarray 库

原来网上有实现啊...
https://github.com/Stack-of-Pancakes/median_cut_color_quantization/blob/master/median_cut.py
https://github.com/Johj/palette/blob/master/palette.js
https://github.com/mvanveen/mcut/blob/master/mediancut.py
https://github.com/tarun98/Median-Cut/blob/master/Median%20Cut.ipynb

将图片内的所有像素加入到同一个区域
对于所有的区域做以下的事：
计算此区域内所有像素的RGB三元素最大值与最小值的差。
选出相差最大的那个颜色（R或G或B）
根据那个颜色去排序此区域内所有像素
分割前一半与后一半的像素到二个不同的区域（这里就是“中位切割”名字的由来）
重复第二步直到你有256个区域
将每个区域内的像素平均起来，于是你就得到了256色
