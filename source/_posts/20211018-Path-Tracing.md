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
&\cdots G(x_{n-1}, x_n) fr(x_{n-1}, x_n, x_{n+1}) \mathbf{d}A_0 \cdots \mathbf{d}A_n
\end{align*}
$$
积分空间实质上是所有长度为$n+1$的光路，光路包括了$n+2$个点。这是一个高维的积分，长度为$n$的路径积分是$2n$维（长度为$n$的路径包含了$n+1$个节点，最后一个节点是固定的即某个像素，下一个顶点可以根据当前顶点和一个方向来选择，三维空间中，方向是一个2维的参数表示，因此，需要选择后续的$n$个顶点，共需要$n$个方向，因此是$2n$维的参数空间）

# 参考

1. [The reandering equation](https://dl.acm.org/doi/10.1145/15886.15902)
2. https://graphics.stanford.edu/courses/cs348b-01/course29.hanrahan.pdf