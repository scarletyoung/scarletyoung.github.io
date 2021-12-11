---
title: Deep Inverse Rendering for High-resolution SVBRDF Estimation from an Arbitrary Number of Images
date: 2021/07/05
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

从一张或多张图片中恢复SVBRDF。

### Difficulty

目前有深度学习的方法可以从一张图片中恢复SVBRDF，但是由于仅有单张图片，无法重建在单张照片中模糊或不可见的反射特征。

但是，还没有深度学习的方法可以处理多张输入图片。

## Contribution

1. 可以处理任意数量的图片，图片数量越多，得到的结果越准确
2. 可以处理任意分辨率的图片，主要是由于auto-encoder可以处理全分辨率的图像。
3. 从单一照片输入中估计的SVBRDFs的质量有所提高，特别是当目标材料不在训练数据集上时。

## Related Work

[Riviere et al. 2016]使用手机和闪光灯记录了一个空间变化的视频样本，使用人工的启发式方法来拟合表面发现和每个点的BRDF，从而识别高光和漫反射。

[Hui et al. 2017]也是使用手机记录视频，迭代地拟合表面发现和基于字典的BRDF模型。

[Palma et al 2012]从在一个在固定自然光照下的物体的视频流恢复空间变化的表面，也是使用一个启发式算法来分别拟合漫反射和高光的反射性质。

[Dong et al. 2014]通过利用入射光线中强边缘的稀疏性，从未知自然光照下的旋转物体视频中估计每个表面点的表面法线和反射率特性。

[Xia et al. 2016]进一步扩展了上述方法，从而恢复物体的形状。

[Aittala et al. 2015]利用每个局部区域在统计上是相似的，从两张图片中（一张有闪光灯一张没有）恢复反射属性

[Xu et al. 2016]也利用了空间关系从两种照片中恢复材质属性。

[Zhou et al. 2016]提出一个通用框架，利用了空间关系和稀疏性，可以恢复SVBRDF从一到多张图片中恢复SVBDDF（不包括法线）。

[Li et al. 2017]使用少量标记的训练数据与大量未标记数据，用于学习一个材料类别特定的卷积神经网络。

[Ye et al. 2018]改进了上述方法，不需要使用标记的的训练数据。

[Deschaintre et al. 2018; Li et al. 2017, 2018]使用深度学习的方法从单张图片中估计SVBRDF。

[Deschaintre et al. 2019]扩展了自己的单张图片方法，可以从多张图片中估计SVBRDF。

[Aittala et al. 2016]从一张照片中估计平面静态材质样本的空间变化的表面发现和反射属性。

## Method

### Deep Inverse Rendering

使用经典的inverse rendering方法，反射参数为$s = (n, k_d, \alpha, k_s)$，损失函数为$\mathcal{L}$，最小化原图像和根据参数渲染的图像的差异，即$\arg \min_s \sum_i \mathcal{L}(I_i, R(s, C_i))$

其中$I_i$是原图像，$C_i$是相机相关的信息。

本文的损失函数使用[Deschaintre et al. 2018]的方法，使用像素值对数的$L_1$距离来表示，即$\mathcal{L}(x, y) = ||\log(x+0.01) - \log(y+0.01)||_1$

本文对传统inverse rendering的改进是，不直接优化参数$s$，而是找到$s$对应的隐空间$z$，通过优化$z$来到达优化s的目的，所以最终的问题变为$\arg\min_z \mathcal{L}(I_i, R(D(z), C_i))$

本文中使用训练的全卷积的auto-encoder来获取参数的latent vector。

### Network Atchitecture

这个网络架构包括两个部分，第一部分是auto-encoder，主要主要目的是训练一个可以从latent vector到参数$s$的decoder。

第二部分是deep inverse rendering框架，用于从latent vector恢复SVBRDF。

### Auto-Encoder Training

训练数据是用[Deschaintre et al. 2018]中的BRDF数据，入是一个$256 \times 256$的反射属性图，中间的latent vector是长度为512的向量。

训练的损失函数是$\mathcal{L}_{train} = \mathcal{L}_{map} + \frac 1 9 \mathcal{L}_{render}$，其中$\mathcal{L}_{map}$是decoder输出的结果和encoder的输入之间的$L_1$ loss，$\mathcal{L}_{render}$是用生成的结果与原始图像的L1 log loss。

使用$\mathcal{L}_{render}$的原因是，在SVBRDF上的微小差异可能会导致渲染图片的明显的差异。

### Optimization Space Smoothness

在latent space中加了一个约束，即$\mathcal{L}_{smooth} = \lambda_{smooth} ||D(z) - D(z + \xi) ||_1$，smoothness在训练的时候，对latent space增加了随机的扰动，用于保证得到的结果相似，使得latent space更加光滑。

### Refinement

全卷积神经编码阶码网络必须会通过一个瓶颈，因此，会损失细节。通常会在encoder和decoder的对应层中增加了连接。

由于这篇论文中主要是从latent vector解码出SVBRDF的参数图，因此无法增加连接。

因此增加了一个后处理步骤，将细节重新引入。

## Advantage

1. 对相机位置没有要求。
2. 可以处理多个图片的输入，单个图片就可以得到看似正确的结果，随着输入图片数量的增加，得到的结果更加准确。
3. 首次在latent code进行优化。

## Limited

1. 需要使用以前的方法来获得一个初始值。
2. 要求至少有一张图片从近似法线的方向拍摄。
3. 需要进行后处理后才能重新一些细节。
4. 输入是$256 \times 256$的，高分辨率的图像需要进行特殊处理。

## My Question

1. refinement怎么进行的？具体似乎要看之间的论文
2. 论文中有个BootStrapping步骤，没有细看。