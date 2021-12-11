---
title: Tracing Ray Differential
date: 2021/08/02
tags: 
- graphics
- rendering
categories:
- note
- grpahics
top: -1
keywords: graphics-rendering-antialiasing
hide: true

---

Ray differentail是光线追踪中的一种高效的antialiasing方法，基本的思路是，当进行光线追踪的时候，不仅追踪一条通过像素点的光线，而且在在像素上方和右方各追踪一条光线，利用这两条光线追踪的信息进行antialiasing。



在光线追踪算法中，从相机到像素$(x,y)$投射一个光线，经过一系列运算得到该像素光谱值v，这一系列运算可以看做是一系列函数，因此有$v=f_n(\cdots f_2(f_1(x, y)))$。

一根光线可以用一个点P和一个方向D表示，即$R=<P, D>$​。在图像坐标空间中，P是视点，D使用view（z轴负向），up（y轴）和right（x轴）表示，即$d=view + y * up + x * right$，对于经过向每个像素投射的光线，可以看做是$(x,y)$​的函数，即$R=r(x, y)$​​。

