---
title: SVBRDF
date: 2021/06/29
tags: 
- graphics
- material
categories:
- note
- grpahics
top: -1
keywords: graphics-material
hide: true
---

# 问题描述

捕获真实世界中的材质属性

几种方法

1. inverse-rendering：定义一个前向渲染模型，优化反射参数，从而使的模拟的外观与现实材质匹配。在使用少量的测量图像时，这个方法是约束不足的，会产生的许多能够匹配的结果，但是大部分是不真实的，无法进行泛化。

   因此，解决方案通常是增加一个预先定义的材质先验来约束优化过程，从而选择出正确的结果。

2. learning-based：从一张或多张图中使用深度学习的方法重建SVBRDF。训练时使用基于渲染的损失，与inverse-rendering类似。测试时，通过深度网络从图像预测SVBRDF。

## Single-Image SVBRDF Capture with a Rendering-Aware Deep Network



## Deep Inverse Rendering

### 目标

目标是使用统一的框架，在单张图片得到可以接受的结果，在多张图片的情况下可以得到准确的结果。

### 假设

假设物体是平坦的，物体表面是使用GGX的微表面模型。

### 基本过程

网络架构是auto-encoder加上一个render层。

网络的输入是SVBRDF，经过encoder得到一个latent vector，然后经过decoder在得到一个SVBRDF，这部分就是原始的auto-encoder，render层使用auto-encoder输出的结果进行图片的渲染。

网络的损失函数同时考虑生成的SVBRDF和渲染的结果，即$\mathcal{L}_{train} = \mathcal{L}_{map} + \frac 1 9 \mathcal{L}_{render}$

其中$\mathcal{L}_{map}$是auto-encoder输出与输出的L1 loss，$\mathcal{L}_{render}$是decoder结果渲染的函数与原本图片的L1 log loss，即$\mathcal{L}_{render}(x,y)=||\log(x+0.01) - \log(y+0.01) ||_1$

使用$\mathcal{L}_{render}$的原因是，在SVBRDF上的微小差异可能会导致渲染图片的明显的差异。系数是$\frac {1} {9}$的原因是$\mathcal{L}_{map}$是9通道的？不过渲染图片的结果是3通道的，这个也不太对吧？

除此之前还加入了latent space光滑的约束，具体是指$\mathcal{L}_{smooth} = \lambda_{smooth}||D(z) - D(z+\xi)||_1$，训练是对latent code中加入噪音后，decoder结果发生的变化不大，加入之后得到的结果更加合理。

当网络训练好的时候，给定一个图片，用其他方法生成一个貌似合理的SVBRDF，生成对应的latent coder，latent coder作为decoder的输入，得到对应的SVBRDF，对图片进行渲染，将渲染的结果与本身图片计算损失函数，使用这个损失函数对latent code进行优化，进行迭代。

初始化使用了[DESCHAINTRE et al. 2018]中的方法，但也可以使用其他的方法。

最后需要对生成的结果进行refinement。原因是auto-encoder在进行卷积的时候会丢失细节。

### 优点和缺点

缺点：

1. 需要初始值，初始值会对最终结果有影响。
2. 需要进行detail refinement得到更细节的信息，得到更好的结果（auto-encoder本身会丢失细节）。
3. 对于复杂的材质需要更多的图片进行恢复。
4. 光源和照片是垂直物体的，可以容忍小偏移。

优点：

1. 在latent space进行优化，
2. 支持全分辨率（auto-encoder本身支持全分辨率的图片）。

## Material GAN

在inverse-rendering框架中提出一个新的材质先验方法MaterialGAN。

MaterialGAN是一个基于SytleGAN2的卷积神经网络，可以获得材质参数的全局相关性，包括了空间上和跨参数的相关性。

通过生成材质映射，来渲染在特定光照和视角下的图片，优化latent向量来使得渲染图片和测试图片之间的误差最小，从而重建相应的材质。

Material GAN的输入是latent space，输出是材质参数。

输入是一个512维的latent向量z，StyleGAN2通过全卷积层的非线性映射网络转换成中间latent向量w，引入w的目的是减少过于严格的约束，产生不太纠缠的映射，有更多有意义的维度。

通过将w重复14次构建一个矩阵w^+^，然后在w^+^中注入了噪声，噪声的目的是生成随机的细节。

最后对w^+^进行优化。

### 优化过程

对于每个图，位置的参数是$\theta = (a, n, r, s)$分别是三通道的环境光，两通道的发现，一通道的粗糙度和三通道的高光，有k个图$I_1, \cdots, I_k$，每个图片有$(L_i, C_i)$，分别代表光照和视角，误差是渲染图片和实际图片的差，所以最终的优化问题为$\theta^* = \arg\min_{a,n,r,s}\sum_{i=1}^k \mathcal{L}(\mathcal{R}(\theta;L_i, C_i), I_i)$

为了利用GAN的先验信息，不是直接优化$\theta$而是优化latent vector，最终的参数通过GAN有latent vector生成。所以优化问题变为$u^* = \arg\min_u \sum_{i=1}^k \mathcal{L}(\mathcal{R}(\mathcal{G}(u);L_i, C_i), I_i)$

损失函数为$\mathcal{I, I'}=\lambda_1 \mathcal{L}_{pixel} + \lambda_2 \mathcal{L}_{percept}$

其中$\mathcal{percept}(I, I') = \sum_{j=1}^4 w_j^{percept}||F_j(I) - F_j(I')||^2_2$

F这里没看明白

仔细想了一下，对于每个材质，$\theta$是不同的，所以应该是不同的优化过程，所以这一步应该是使用MaterialGAN生成材质信息的过程，而不是训练GAN的过程。

没有弄明白gan的训练过程和latent vector的生成过程。

### 基本步骤

类似于上面的方法，将auto-encoder用MaterialGAN代替。

输入是一个512维的latent向量z，StyleGAN2通过全卷积层的非线性映射网络转换成中间latent向量w，引入w的目的是减少过于严格的约束，产生不太纠缠的映射，有更多有意义的维度。

通过将w重复14次构建一个矩阵w^+^，然后在w^+^中注入了噪声，噪声的目的是生成随机的细节。

所以实际的参数是w^+^和$\xi$，论文测试了多种优化方案，最终决定交替优化w^+^和$\xi$可以获得更好的结果。



### 优点和缺点

缺点：

1. 需要知道相机和光源的位置
2. 对于多层材质无法很好的重建
3. 当高光非常小的时候，效果不好，需要更多的图片。

优点：

