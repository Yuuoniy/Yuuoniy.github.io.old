---
title: cv | CImg 基础-图像读取以及像素操作
date: 2018-09-13 13:03:25
tags:
---
首先下载库，
根据参考文档，学习一些基础的函数，[中文文档](http://files.cppblog.com/coreBugZJ/CImg%E5%BA%93%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C%E4%B8%AD%E6%96%87%E7%89%88_v_1_0_0.pdf)
[github地址](https://github.com/dtschump/CImg) 包含一些样例，可以参考
现根据问达能写简单的代码测试一下
```c++
#include "CImg.h"
using namespace cimg_library;
int main()
{
  CImg<unsigned char> img(640, 400, 1, 3);        // Define a 640x400 color image with 8 bits per color component.
  img.fill(0);                                    // Set pixel values to 0 (color : black)
  unsigned char purple[] = {255, 0, 255};         // Define a purple color
  img.draw_text(100, 100, "Hello World", purple); // Draw a purple "Hello world" at coordinates (100,100).
  img.display("My first CImg code");              // Display the image in a display window.
  return 0;
}
```
编译：`g++ 1.cpp -O2 -lgdi32` ,运行可以看到紫色的helloworld
基础操作：
1. 读入1.bmp文件，并用CImg.display() 显示
```c++
 CImg<unsigned char> SrcImg;        // Define a 640x400 color image with 8 bits per color component.
  SrcImg.load_bmp("1.bmp");
  int w = SrcImg._width;
  int h = SrcImg._height;
  CImg<unsigned char> TempImg(w,h,1,1,0);
  //显示图像
  SrcImg.display();
```

2. 把 1.bmp 文件的白色区域变成红色，黑色区域变成绿色

3. 在图上绘制一个圆形区域，圆心坐标(50,50)，半径为 30，填充颜色为蓝色。
```c++
CImg<T>& draw_circle	(	const int 	x0,
const int 	y0,
int 	radius,
const tc *const 	color,
const float 	opacity = 1 
)	
```
4. 在图上绘制一个圆形区域，圆心坐标(50,50)，半径为 3，填充颜色为黄色。
5. 在图上绘制一条长为 100 的直线段，起点坐标为(0, 0)，方向角为 35 度，直线
的颜色为蓝色。
6. 把上面的操作结果保存为 2.bmp。
const unsigned char blue[] = {0, 0, 255};
