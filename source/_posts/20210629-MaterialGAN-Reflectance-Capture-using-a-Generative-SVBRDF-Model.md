---
title: MaterialGAN: Reflectance Capture using a Generative SVBRDF Model
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

从几张从图片中重建SVBRDF。

### difficulty

在使用机器学习的方法的时候，通常需要大量的数据。在少量数据的情况下，这个问题是约束不足的，会产生大量在训练集上的可行解，但无法泛化到新的光照和视角。

## Contribution

为了解决存在的困难，需要在训练的时候加入一个先验，从而对问题的解提供一些约束，使得最终可以获得纬一街。

本文使用GAN训练一个先验，加入到inverse rendering-based的材质捕获框架中，

## Related Work

[Hui et al. 2017]，[Xu et al. 2016]，在材质上增加了一个强的先验。

[Aittala et al 2016, 2015]从two-shot和one-shot图片中估计静态SVBRDF的每像素参数。

[Deschaintre et al. 2018; Li et al. 2017, 2018]使用深度学习的方法从单张图片中估计SVBRDF

[Deschaintre et al. 2019]扩展了自己的单张图片方法，可以从多张图片中估计SVBRDF。

[Gao et al. 2019]训练auto-encoder网络学习latent space，作用材质的先验，并且在latent space上进行优化。

## Method

本文是在[Gao et al. 2019]的基础上进行了改进，将auto-encoder替换为了GAN，使用GAN的结果作为先验，对结果进行约束。

GAN相较于auto-encoder的优点是可以获得材质参数的全局相关性，包括了空间上和跨参数的相关性。

### Introduction of MaterialGAN

MaterialGAN是在SytleGAN2上进行改进的，StyleGAN2的结构可以参考[StyleGAN 和 StyleGAN2 的深度理解][https://zhuanlan.zhihu.com/p/263554045]

StyleGAN2的输入是隐空间$z \in \mathcal{Z} \subset \mathbb{R}^{512}$，通过一系列仿射变换将z转换到$w \in \mathcal{W} \subset \mathbb{R}^{512}$，转换的理由是z是对应于一个输出，w可以放宽这个约束，然后将w送到每个卷积的前后。

MaterialGAN的改进如下

通过复制w构建了矩阵$w^+ \in \mathcal{W}^+ \subset \mathbb{R}^{512 \times 14}$，在StyleGAN2中每层的w是相同的，MaterialGAN中可以放松了这个限制。StyleGAN2中在每层加入了高斯噪声$\xi \in N$，因此，最终的输入空间是$\mathcal{W^+N}$

加入噪声的目的是为了生成更多的细节

### MaterialGAN Training

因为对GAN不是很熟悉，对这里不是很理解。

MaterialGAN的输入应该是latent vector，输出是一个九通道的图，三通道的albedo，两通道的法向量，一通道的粗糙度和三通道的specular albedo。

训练数据是[Deschaintre et al. 2018]中提供的数据。

输入latent vector，将网络生成的九通道图与训练数据的九通道图进行比较。

训练数据的latent vector怎么得到的，用auto-encoder吗？

### Capture Material Overview

在inverse rendering-based框架中，材质模型是使用GGX法线分布的微表面模型，因此，BRDF参数为$\theta = (a, n, r, s)$。

学习的目标是，使用学习的BRDF进行渲染得到的结果与捕获的图片一致，设捕获的图片为$I_1, \cdots, I_k$，每个图片的光照和视角为$(L_i, C_i)$，因此最终的学习目标是$\theta^* = \arg \min_{a, n, r} \sum_{i=1}^k \mathcal{L}(\mathcal{R}(\theta;L_i, C_i), I_i)$。

在MaterialGAN作为先验加入到框架中之后，参数latent vector输入MaterialGAN中，得到BRDF参数$\theta$，$u^* = \arg \min_{w^+, \xi} \sum_{i=1}^k \mathcal{L}(\mathcal{R}(\mathcal{G}(w^+, \xi);L_i, C_i), I_i)$。

### Loss Function

损失函数是per-pixel的L2损失和渲染结果的差异，$\mathcal{L}(I, I')=\lambda_1 \mathcal{L}_{pixel} + \lambda_2 \mathcal{L}_{percept}$

其中$\mathcal{L}_{percept}(I, I') = \sum_{j=1}^4 w_j^{percept}||F_j(I) - F_j(I')||^2_2$，在[Johnson et al. 2016]中提出。

其中F是预训练的VGG网络的各层的输出，这里有点不明白，可能需要看下[Johnson et al. 2016]

### Optimazation

有两个参数需要优化，$w^+$和$\xi$。实验表明，每次交替优化$w^+$和$\xi$几步可以得到较好的结果。同时在VGG中，$w^+$和$使用不同的权重。

## Advantage

1. 与[Gao et al. 2019]等人的方法相比，不需要特定初始值，随机的初值也可以得到很好的结果。
2. 与[Gao et al. 2019]等人的方法相比，能够还原更加材质的细节。
3. 与[Gao et al. 2019]等人的方法相比，即使不进行改善，也可以得到较好的结果。
4. 可以通过latent vector进行插值。

## Limited

1. 与之前的方法一样，假设材质是平坦的。
2. 渲染模型和BRDF模型是简单的，无法描述复杂的材质和复杂的反射情况，如多层材质。
3. 强烈的自阴影和相互反射也没有处理。
4. 无法描述各项异性的材质
5. 输入是$256 \times 256$？这个不确定，似乎是可以通过上采样和下采样来得到高分辨率的SVBRDF

## My Question

1. 训练用的的latent vector是如何得到的？
2. VGG网络在什么时候用的，目的是什么？
3. SVBRDF和BRDF是什么关系？