---
title: Importance Sampling
date: 2021/09/30
tags: 
- graphics
- rendering
categories:
- note
- grpahics
- sampling
top: -1
keywords: graphics-rendering-sampling
---

# 蒙特卡洛积分

根据渲染方程，在点p的着色为

$$L_o(p, w_o) = L_e(p, w_o) + \int_{\Omega^+} L_i(p, w_i)fr(p, w_o, w_i) (n \cdot w_i) \mathbf{d} w_i$$

因为积分形式存在递归且被积函数复杂，因此难以求解。

为了获取积分结果，通常使用蒙特卡洛方法进行数值求解。蒙特卡洛方法通过在积分域上随机选择一些点来对积分进行估计。具体定义为，对于积分$I = \int_\Omega f(x) dx$，根据分布$p(x)$从x上采样$N$个点$x_1, \cdots, x_n$，则积分$I$可以近似为$I \approx Q = \frac 1 n \sum_{i=1}^N \frac {f(x_i)} {p(x_i)}$。

对$Q$求期望，可得

$$\mathbb{E}(Q) &= \mathbb{E}(\frac 1 n \sum_{i=1}^n \frac {f(x_i)} {p(x_i)}) \\& = \frac 1 n \sum_{i=1}^N \mathbb{E}(\frac {f(x_i)} {p(x_i)}) \\ &= \int_a^b \frac {f(x)} {p(x)} p(x) \mathbf{d} x$$

从上式可知，蒙特考虑积分$Q$的对积分$I$的无偏估计。

因此，对于间接光照部分（积分部分），根据蒙特卡洛积分可以得$L_{indir} = \frac 1 N \sum_{i=1}^N \frac { L_i(p, w_i)fr(p, w_o, w_i) (n \cdot w_i)} {p(w_i)}$，其中$w_i$从分布$p(w)$中采样。

# 重要性采样

上一节中已经证明了蒙特卡洛积分的期望等于积分的解，因此，通过采样一定数量的样本可以求得积分的近似解。那么，现在的问题在于，需要多少样本才能最终得到可接受的近似解。

考虑一个简单的例子（例子和数据来源于<https://zhuanlan.zhihu.com/p/360420413>）

一个简单的积分$I = \int_0^4 x dx$，直接求解该积分可以得到结果为8，使用四种不同的pdf来对此积分进行近似求解，有如下结果。

|      采样函数      | 0.008标准差所需要的样本数 |
| :----------------: | :-----------------------: |
| $\frac {6-x} {16}$ |          887,500          |
|    $\frac 1 4$     |          332,812          |
| $\frac {x+2} {16}$ |          98,437           |
|    $\frac x 8$     |             1             |

从上表中可以看出，收敛所需的样本数随着采样函数的不同而不同，因此选择合适的分布可以显著减少收敛所需要的样本数量。

那么如何选择合适的$p(x)$？目标是使用尽量少的样本减少方差，因此对蒙特卡洛积分求方差有$\sigma^2(Q) = \sigma^2(\frac 1 n \sum_{i=1}^n \frac {f(x_i)} {p(x_i)}) = \frac 1 {n^2} \sigma^2(\sum_{i=1}^n \frac {f(x_i)} {g(x_i)})$

因为，样本$x_1, \cdots, x_n$是独立同分布，所以，可以化简为$\sigma^2(Q) = \frac 1 {n^2} \sum_{i=1}^n \sigma^2(\frac {f(x_i)} {p(x_i)}) = \frac 1 n \sigma^2(\frac {f(x)} {p(x)})$

从上式可以看出，蒙特卡洛积分$Q$的方差取决于$\frac {f(x)} {p(x)}$的方差，因此，当$p(x)$和$f(x)$的形状相似，方差近似为常数，即当$p(x) = cf(x)$的时候方差最小。



根据上面的结论，$p(x)=cf(x)$时方差最小，问题在于，在路径追踪者无法得到$f(x)=L_i(p, w_i)fr(p, w_o, w_i) (n \cdot w_i)$的解析式，无法求得最优的$p(x)$，因此，将条件放松，使得$p(x)$和部分$f(x)$成正比，以减少需要采样的次数。根据$f(x)$可知，想要$p(x)$部分正比于$f(x)$，可以使$p(x)$正比与$L$，$pbrt$或$cos$项。

### 对cos项进行采样

对$cos$项进行采样时，有$p(w)=c \cdot \cos \theta_i$，根据$\int p(w) dw = 1$可解得$c = \frac 1 \pi$，因此$p(\theta, \phi) = \frac {\cos\theta \sin \theta} {\pi}$

再求出$p(\theta)$的边缘概率分布以及$p(\phi|\theta)$以及对应的概率密度函数，

$$p(\theta) = \int_0^{2 \pi} \frac {\cos\theta \sin\theta} {\pi} \mathbf{d}\phi = \sin2\theta \\ p(\phi|\theta) = \frac {p(\theta, \phi)} {p(\theta)} = \frac 1 {2 \pi} \\ F(\theta) = \int_0^\theta p(\theta) \mathbf{d}\theta = \int_0^{\theta} \sin2\theta \mathbf{d}\theta = \sin^2\theta \\ F(\phi|\theta) = \int_0^{\phi} p(\phi|\theta) \mathbf{d} \phi = \int_0^{\phi} \frac 1 {2\pi} \mathbf{d} \phi = \frac {\phi} {2\pi}$$

设$\xi_1, \xi_2$是$[0,1]$的随机数，则采样结果为$\theta = \arcsin \sqrt {1 - \xi_1}, \phi=2\pi\xi_2$，利用球坐标相关的知识可以得到三维空间的采样坐标。

### 对BRDF进行采样

当使用微表面模型来描述物体BRDF时，有$f_r(w_i, w_o) = \frac {D(w_h) G(w_i, w_o, w_h) F(w_i \cdot w_H)} {4 \cos\theta_i \cos\theta_o}$，由于$G$和$F$项是在采样得到$w_i$后才能进行计算，因此，仅使的$p(w)$与$D(w)$相似。

在Cook-Torrance BRDF推导过程中，可知$\int \cos\theta_h D(w_h) \mathbf{d}w_h = 1$和$\frac {\mathbf{d}w_h} {\mathbf{d}w_i} = \frac 1 {4 (w_h \cdot w_i)}$，因此有$\int \frac {\cos\theta_h D(w_h)} {4 (w_h \cdot w_i)} \mathbf{d} w_i = 1$，因此被积函数就是采样函数。

与cos项采样类似，最终也可以得到三维空间的采样坐标。

# 多重重要性采样

当渲染面光源在光滑表面的高光效果时，必须有一条从相机到光源的光路才能对最终的像素有贡献。

此时，若对BRDF进行重要性采样，即$p(w)$分布近似于BRDF，此时，采样的入射光线$w_i$必须与光源相交才能对最终结果有贡献。此时，会有大量的样本被浪费（即无法采样到光源）。这个情况下，直接在面光源上进行采样效率会比较高（每个采样的光线都对最终结果有贡献）。上面两种策略就是常用的两种采样策略，采样BRDF和采样光源。

考虑下图中的两个例子



当光源面积小的时候，对brdf的采样时，采样光线很难与光源相交，尤其是表明较为粗糙的时候。而当光源面积大的时候，对光源采样时，在光滑表面上，该方向的brdf不存在或很小，因此在光源表面上效果很差。两个采样方法在一种情况效果较好，但在另一种情况下效果较差。因此，为了结合多种采样方法的优点，Veach提出了多重重要性采样（MIS）。

MIS的定义如下，设有$n$个采样方法$p_1, \cdots, p_n$，从$p_i$中获取的相对样本数量为$c_i$，其中$\sum_{i=1}^nc_i=1$，设总共采样$N$个样本，从每个分布$p_i$中采样$n_i = c_i N$个样本，则对积分的最终估计为$Q = \sum_{i=1}^n \frac 1 {n_i} \sum_{j=1}^{n_i} w_i(X_{i,j})\frac {f(X_{i,j})} {p(X_{i,j})}$，其中$w_i(x)$是采样方法$p_i$的权重且$\sum_{i=1}^n w_i(x) = 1$

简单来说，多重重要性采样就是将多个采样方法得到的结果进行加权平均。

多重重要性采样的一个问题就是如何选择权重，作者在文中提出了几种方法

## balance heuristic

$w_i$的表达式为$w_i(x) = \frac {c_i p(x)} {\sum_{j=1}^n c_j p_j(x)}$

理论显示，balance heuristic是一个足够好的权重，但仍然有优化的空间。

考虑下图中的两个采样函数，其中$p_1$完美地与$f$成比例，而$p_2$是一个均匀分布。此时，理想的权重是$w_1(x)=1$且$w_2(x)=0$。

![image-20211013155543606](/Users/shellingford/Library/Application Support/typora-user-images/image-20211013155543606.png)

通常，由于$p$和$f$都没有解析式，无法确定这个情况，但是，通过选择合适的权重来近似。通过分析可知，当$p_1$的样本远离$f$的峰值时，$p_1$样本权重是$\frac {p_1} {p_1 + p_2}$，比理想的结果较小。当$p_2$的样本靠近$f$的峰值时，此时$p_2$的权重非常小，使得最终的贡献也非常小。因此提出了以下两种启发式方法，使得大权重更大，小权重更小。

## cutoff heuristic

cutoff heuristic基本思路是丢弃小权重的样本，表达式为

$$w_i = \begin{cases} 0 & p_i \lt \alpha p_{max} \\ \frac {p_i} {\sum_j \{p_j | p_j \ge \alpha p_{max}\}} & otherwise \end{cases} $$

其中$p_{max}=\max {p_j}$，$\alpha$来控制多小权重的样本应该被丢弃。

## power heuristic

power heuristic的思路是将权重乘方后再进行归一化，表达式为

$$w_i = \frac {p_i^\beta} {\sum_j p_j^\beta}$$

# 连续多重重要性采样



# 参考

























