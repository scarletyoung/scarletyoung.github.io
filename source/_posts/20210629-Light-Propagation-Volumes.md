---
title: Light Propagation Volumes
date: 2021/06/29
tags: 
- graphics
- RTRT
- GI
categories:
- note
- grpahics
top: -1
keywords: graphics-RTRT-LPV
hide: true
---

# Light Propagation Volumes

渲染方程的解为

$I = (1-KG)^{-1} \circ Ge_0 = (1 + KG + (KG)^2 + \cdots) \circ Ge_0$

其中I是光强，$e_0$是光源发射的光，G是可见性算子，K是线性局部反射算子。

因为只考虑一次间接光照，所以，要解的公式为$I = I_0 + I_1 + I_2$

$I_0=G_0e_0$是光源直接对像素的贡献，$G_0$是光源的可见性，使用depth buffer处理

$I_1=G_0K_1G_1e_0$是光线经过一次弹射对像素的贡献，其中$K_1$是光线在物体表面的反射算子，$G_1$是光源到物体的可见性，通常使用shadow maps进行处理。

$I_2 = G_0 K_1 G_1 K_2 G_2 e_0$是经过两次弹射的光线对像素的贡献，其中$K_2G_2$是光线第一次在物体反射的结果，为了简化处理，通常这次反射被认为是纯漫反射。其中$K_2G_2e_0$也认为是在没有可见性的$I_1$，像是reflective shadow map就记录了$I_1$，利用$I_1$计算一次间接光照。



算法分为四个步骤

1. 将场景渲染为reflective shadow map来生成次级光源的集合。
2. 用三维纹理来表示一个radiance field，纹理存储的值是球谐系数，将虚拟光源转换为球谐系数的表示形式，将这个值写入到该光源在三维纹理中对应的位置中。
3. 通过迭代求解，得出每个三维纹理中的radiance。
4. 将最终的三维纹理应用到场景照明中，

## Scene point cloud generation

使用RSM生成次级光源





## reference

1. Light Propagation Volumes in CyrEngine 3