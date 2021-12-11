---
title: Flexible SVBRDF Capture with a Multi-Image Deep Network
date: 2021/07/06
tags: 
- graphics
- material
- SVBRDF
categories:
- note
- grpahics
- material
- paper
top: -1
keywords: graphics-material
hide: true
---

## Problem

利用深度学习的方法从少数几张图片中恢复出物体的材质信息。

### difficulty

大部分的深度网络要求固定数量的图片，并且图片的顺序会影响结果。

## Contribution

## Related Work

[WSM11, AWL15,AAL16,RWS11,WDT10,HSL17]从少量的图片中利用关于材质的强先验来恢复SVBRDF。

[LDPT17, YLD18, DAD18, LSC18]利用深度学习的方法从平摊材质的单张图片中恢复出可以接受的SVBRDF。

[RPG 16, HSL 17]也是基于优化的方法，但需要较多的图片。

[Wiles et al. 2017]使用多个编码器处理每个图像，然后通过一种顺序无关的操作来合并特征。

## Method

本论文中使用了的网络有多个单图像网络的副本组成。单个网络的数量是根据用户输入的图像动态选择的。多个网络的结构和权重是完全相同的。

单个网络结构使用的是作者在18年发表的论文中提出的方法，将最后一层做了修改，不是输出4个SVBRDF图，而是输出64通道的特征图。像素坐标作为额外通道提供。

每个网络生成$256 \times 256 \times 64$的特征图，这些特征图通过在每个像素和特征通道选择最大的值了合成单个特征图。这个max-pooling程序使得每个图片的有均等的方式对最终结果做出贡献。

汇聚的特征图由三层卷积和非线性解码。

损失函数是$L = L_{render} + 0.1 * (L_{Normal} + L_{Diffuse} + L_{Specular} + L_{Roughness})$

## Advantage

1. 可以从少量图片中恢复出较高质量的BRDF信息。

## Limited

1. 使用的模型是Cook-Torrance BRDF模型，无法得到该模型无法表示的材质信息
2. 没有对阴影投影进行建模。

## My Question

