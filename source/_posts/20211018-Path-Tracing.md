---
title: Path Tracing
date: 2021/09/30
tags: 
- graphics
- rendering
categories:
- note
- grpahics
- rayTracing
top: -1
keywords: graphics-rendering-rayTracing
---

# 渲染方程

渲染方程为
$$
L(x, w_o) = L_e(x, w_o) + \int_{\Omega^+} L(x, w_i) fr(x, w_i, w_o) cos\theta_i \mathbf{d} w_i
$$
其中

+ $L(x, w_o)$表示点$x$向立体角$w_o$方向辐射的radiance。

+ $L_e(x, w_o)$表示点$x$自身向立体角$w_o$方向辐射的radiance。

+ $fr(x, w_i, w_o)$是散射函数（BRDF），表明从$w_i$方向入射到$x$的radiance有多少从$w_o$的方向射出。积分部分表示点$x$从其他物体接收的光照中，向立体角$w_o$方向辐射的radiance。

因此，渲染方程的含义是，点$x$向方向$w_o$的radiance等于物体自身向$w_o$方向的radiance和所有从半球面方向$w_i$接收的到raidnance散射到$w_o$方向的radiance之和。

# 路径积分

为了更加直观的表示渲染方程，我们使用点的形式对渲染方程进行改写，首先，使用$w(x,x')$来表示两个点之间的方向，即$w(x,x') = w(x \to x') \frac {x' - x} {||x' - x||}$。

因此，我们可以用点的形式来对radiance和BRDF进行重新表示，即$L(x, x') = L(x \to x') = L(x, w(x, x'))$和$fr(x, x', x'') = f(x \to x' \to x'') = fr(x', w(x', x), w(x', x''))$。

当使用点的形式来表示渲染方程的时候，积分微元也必须做相应的改变，对于两个点的几何关系，如下图所示

![image-20211019112544568](/Users/shellingford/Library/Application Support/typora-user-images/image-20211019112544568.png)

根据立体角与积分的关系，可以得出$\mathbf{d}w_i = \frac {\cos\theta'} {||x-x'||^2} \mathbf{d}A(x')$

由于积分由立体角方向变为了点，需要考虑两个点之间的可见性，因此需要引入可见性函数$V(x,x')$，当两个点$x$和$x'$相互可见时，$V(x, x')=1$，否则$V(x, x')=0$。

因此，$\cos \theta_i \mathbf{d} w_i = G(x, x')\mathbf{d} A(x')$，其中$G(x, x') = \frac {\cos \theta_i \cos \theta'} {||x - x'||^2} V(x, x')$。

因此，渲染方程可以改写为另一种形式
$$
L(x, w) = L_e(x, w) + \int_{A(x)} L(x', w(x',x))fr(x, w(x,x'), w) G(x, x') \mathbf{d} A(x')
$$
从渲染方程的形式可以看出，求解渲染方程是一个递归的过程。

将积分作为算子，可以将渲染方程写成更简单的算子的形式，即$L = L_e + K \circ L$。（这里完全不明白为什么）

递归过程可以通过迭代求解，即

$L^0 = L_e$，此时表示物体的radiance等于物体自身散发的radiance

$L^1 = L_e + K \circ L^0 = L_e + K \circ L_e$，此时表示物体的radiance等于物体自身散发的radiance加上直接光照的radiance。

$L^n = L_e + K \circ L^n-1 = \sum_{i=0}^n K^n \circ L_e$，表示物体的的radiance等于物体自身散发的radiance，直接光照的radiance和$n-1$次间接光照的radiance之和，这里为了简便，令$I = K^0$

可以看出，$\sum_{i=0}^\infty K^i$是一个Neumann级数，因此，根据Neumann级数的属性有$(I-K)^{-1} = \sum_{i=0}^\infty K^i$。

根据渲染方程的算子形式，可以解得$L = (I-K)^{-1}L_e$，因此，有$L = (I+K+K^2 \cdots ) \circ L_e$



对于其中一项$L^n$，表示从光源经过$n$次反射或折射后对某个像素的贡献，将其展开为积分形式有
$$
\begin{align*}
L^n(x, \omega) &= L(x_{n}, x_{n+1}) =  K^n \circ L_e \\
&= \int_{A_n} \cdots \int_{A_0} L_e(x_0, x_1) G(x_0, x_1) fr(x_0, x_1, x_2) G(x_1, x_2) fr(x_1, x_2, x_3) \\
&\cdots G(x_{n-1}, x_n) fr(x_{n-1}, x_n, x_{n+1}) G(x_n, x_{n+1}) \mathbf{d}A_0 \cdots \mathbf{d}A_n
\end{align*}
$$
积分空间实质上是所有长度为$n+1$的光路，光路包括了$n+2$个点。这是一个高维的积分，长度为$n$的路径积分是$2n$维（长度为$n$的路径包含了$n+1$个节点，最后一个节点是固定的即某个像素，下一个顶点可以根据当前顶点和一个方向来选择，三维空间中，方向是一个2维的参数表示，因此，需要选择后续的$n$个顶点，共需要$n$个方向，因此是$2n$维的参数空间）



上述的公式看起来非常复杂，为了得到更简洁的表达式，需要引入几个符号。

令$\bar{x}_k=x_0 x_1 \cdots x_k$表示一条长度为$k$的光路，其中$1 \le k \lt \infin$且$\forall x_i \in \mathcal{M}$。$\mathcal{M}$表示场景中所有表面点的集合。令$\Omega_{k}$表示长度为$k$的光路集合，$\Omega = \cup_{i=0}^{\infin} \Omega_i$表示光路空间，即所有有限长度的光路。

在光路集合$D \sub \Omega_i$上定义面积乘积度量$\mu_k = \int_D \mathbf{d} A(x_0) \cdots \mathbf{d}A_(x_k)$。因此，有$\mathbf{d}\mu_k = \mathbf{d} A(x_0) \cdots \mathbf{d}A_(x_k)$

将被积部分定义为$f_i(\bar{x}) = L_e(x_0, x_1) G(x_0, x_1) fr(x_0, x_1, x_2) G(x_1, x_2) fr(x_1, x_2, x_3) \cdots G(x_{n-1}, x_n) fr(x_{n-1}, x_n, x_{n+1}) G(x_n, x_{n+1})$，$f_i(\bar{x})$称为度量贡献函数。

最终，路径积分可以用以下表达式表达
$$
I_j = \int_\Omega f_j(\bar{x}) \mathbf{d}\mu(\bar{x})
$$

## 计算路径积分

为了计算路径积分$I_j = \int_\Omega f_j(\bar{x})\mathbf{d}\mu \bar{x} $。使用蒙特卡洛方法来对积分进行估计，使用概率密度函数$p$进行采样得到路径$\bar{x}$，则积分近似值为$I_j \approx \frac {f_j(\bar{x})} {p(\bar{x})}$。这个估计是无偏的，证明见[重要性采样](20210930-Importance Sampling.md)

为了得到估计值，需要计算$f_j$和$p$，其中$f_j$在上文中已经得知，重点在于如何计算得到$p$。路径采样的概率不仅与采样的点有关，还与路径采样的概率有关。

一个简单的路径生成算法是局部路径采样算法（local path sampling algorithm），这个算法有三个机制

1. 在面光源上或透镜上根据一些先验分布在场景表面进行采样
2. 在现在有的点上，根据当前点的BSDF对方向进行采样，投射光线与场景相交得到另一个点。
3. 连接两个现有的点，测试他们的可见性。

最终，生成路径的过程为，从光源、透镜或其他场景表面选择一个点和方向，投射光线生成子路径，将子路径之间连接起来形成一个从光源到相机的完整路径。



在路径积分中，积分项是面积乘积度量$\mu(\bar{x})$，因此，概率密度函数是关于$\mu(\bar{x})$的，即$p(\bar{x}) = \frac {\mathbf{d}P} {\mathbf{d} \mu} \bar{x} = \frac {\mathbf{d}P} {\mathbf{d}\mu} (x_0, \cdots, x_k) = \prod_{i=0}^k \frac {\mathbf{d}P} {\mathbf{d}A}(x_i)$。为了评估概率密度函数$p(\bar{x})$，因此，需要计算$\frac {\mathbf{d}P} {\mathbf{d}A} (x_i)$。



根据局部路径采样算法，生成点的方式有两种

1. 根据表面的分布进行采样，这种方式可以直接计算。

2. 随机选择一个方向$w_o$，从现有的点$x$投射一根光线，得到交点$x'$。

   可以直接得到的密度是$\frac {\mathbf{d}P} {\mathbf{d}\sigma} (\omega)$，是关于立体角的概率密度函数，而需要计算的是关于面积的概率密度函数，根据二者的关系$\frac {\mathbf{d} P} {\mathbf{d} A} (x_i) = \frac {\mathbf{d} P} {\mathbf{d} \sigma} (\omega) \frac {\mathbf{d}\sigma(w)} {\mathbf{d} A(x)}$，可得$p(x') = p(\omega) (\frac {|\cos \theta_i'|}{||x-x'||^2})$，在计算中，通常使用投影立体角而不是立体角，根据二者的关系$p^\perp(\omega) = \frac 1 {\cos \theta_o} p(\omega)$，因此，最终的概率密度表达式为$p^\perp(x') = p^\perp(w_o) \frac {|\cos \theta_o \cos \theta_i'|} {||x-x'||^2} = p^\perp(w_o) G(x, x')$

   





# 参考

1. [The reandering equation](https://dl.acm.org/doi/10.1145/15886.15902)
2. https://graphics.stanford.edu/courses/cs348b-01/course29.hanrahan.pdf