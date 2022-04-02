---
title: Continuous Multiple Importance Sample
date: 2022/03/13
---

# 多重重要性采样的问题

在蒙特卡洛路径追踪中，路径采样的计算量很大，占据了大部分的运算时间，一个自然的优化方法是在多个像素中重复使用采样的子路径。

路径重用可以描述如下，像素的值$I$可以表示为在从视点到光源的所有可能光传输路径空间$\mathcal{P}$上的积分，即

$$
I = \int_{\mathcal{P}}f(\bar{x}) \mathbf{d} \bar{x}
$$

其中，$\bar{x} = x_1x_2 \cdots$表示一条从视点到光源的光路。

对于一条光路$\bar{x}$，可以分解为前缀子路径$\bar{y}$和后缀子路径$\bar{z}$，即$\bar{x} = \bar{y}\bar{z}$，同时，可以将路径对像素的能量共享分解为$f(\bar{x}) = f_e(\bar{y})f_l(\bar{y}, \bar{z})$，其中$f_e(\bar{y})$是从视点开始的光路的贡献，$f_l(\bar{y}, \bar{z})$是来自光源的贡献，其中还包含子路径$\bar{y}$最后一个节点和子路径$\bar{z}$的第一个节点之间的可见性。

根据上述分解，积分可以重写为

$$
I = \int_{\mathcal{P}}f(\bar{x}) \mathbf{d} \bar{x} = \int_{\mathcal{P}}f_e(\bar{y}) \int_{\mathcal{P}}f_l(\bar{y}, \bar{x}) \mathbf{d} \bar{z} \mathbf{d} \bar{y} = \int_{\mathcal{P}}f_e(\bar{y}) I(\bar{y}) \mathbf{d} \bar{y}
$$

在使用路径重用时，使用采样的一个前缀子路径$\bar{y}$和$n$个后缀子路径$\bar{z}_i$对积分结果进行估计，即

$$
<I> = \frac {f_e(\bar{y})} {p(\bar{y})} <I(\bar{y})>_n = \frac {f_e(\bar[y])} {p(\bar{y})} \sum_{i=1}^n w(\bar{y}, \bar{z}_i) \frac {f_l(\bar{y}, \bar{z}_i)} {p(\bar{z}_i)}
$$

在路径重用中，我们将每个路径$\bar{x}$的前缀$\bar{y}$和其他$n$个路径的$\bar{x}_i=\bar{y}_i \bar{z}_i$连接起来，从而达到路径重用的目的。然而，上述积分无法使用多重重要性采样进行估计，原因有两个

1. 每个路径后缀$\bar{z}_i$有不同的前缀，因此$\bar{z}_i$时根据不同的分布采样而来的，即$\bar{z}_i \sim p(\bar{z}_i|\bar{y}_i)$
2. 每个路径后缀$\bar{z}_i$可能的前缀$\bar{y}_i$时一个不可数无限集。

现有的一些路径重用方法都存在一些局限性，因此，需要一个方法能够从连续空间中组合抽样技术问题，这就是连续多重重要性采样。

# 连续多重重要性采样

## 连续多重重要性采样公式

使用$\tau$表示一个任意维度空间，允许按照一定的PDF进行采样，称$\tau$为技术空间，空间中的每个元素$t$确定了一个采样技术

还引入了连续加权函数$w: \tau \times \chi \to \mathbb{R}$，加权函数$w$的空间$\tau$上积分恒为一，即$\int_\tau w(x,t) \mathbf{d} t = 1$，其中$\mathbf{d}t$是在空间$\tau$上适当的微分度量。

根据上述的定义，可以将重要性采样的公式进行扩展

$$
I = \int_\chi \int_\tau w(t, x) \mathbf{d} t f(x) \mathbf{d} x = \int_\tau \int_\chi w(t, x) f(x) \mathbf{d} x \mathbf{d} t
$$

公式中的加权函数积分为一，与多重重要性采样公式中的求和为一对应。

根据上述公式，单样本的连续多重重要性采样(CMIS)估计器为

$$
<I>_{CMIS} = \frac {w(t,x) f(x)} {p(t,x)} = \frac {w(t, x) f(x)} {p(t) p(x|t)}
$$

其中，$t$和$x$是根据联合分布概率$p(t,x)$采样得到的连续随机变量，当变量$x$依赖于$t$的时候可以分解为$p(t,x) = p(t)p(x|t)$。$p(t)$表示选择技术$t$的概率，$p(x|t)$是条件概率。这个估计器于多重重要性采样的单样本估计器相似。

为了使得估计是无偏的，加权函数必须满足以下条件

$$
\begin{cases}
\int_\tau w(t, x) \mathbf{d} t = 1 & f(x) \neq 0 \\
w(t, x) = 0 & p(t,x) = 0
\end{cases}
$$

在估计是无偏的前提下，我们希望估计的方差尽可能小，从而可以在少量样本的情况下达到收敛状态，减少计算时间，因此我们希望可以选择估计偏差最小的加权函数$w$，根据推导，可以得知最有的加权函数为

$$
\bar{w}(t,x) = \frac {p(t)p(x|t)} {\int_\tau p(t')p(x|t')\mathbf{d}t'} = \frac {p(t, x)} {\int_\tau p(t', x) \mathbf{d}t'} = \frac {p(t,x)} {p(x)}
$$

其中$p(x)$是$x$的边缘概率密度函数。

将上述加权函数带入CMIS估计器中可以得到

$$
<I>_{CMIS} = \frac {\bar{w}(t,x)f(x)} {p(t,x)} = \frac {p(t,x)f(x)} {p(x)f(t,x)} = \frac {f(x)} {p(x)}
$$


## 随机多重重要性采样公式

在上述公式的计算中，我们需要评估边缘概率密度函数$p(x)$，这个积分通常是没有解析解的。作者使用了一个简单的方法来近似。

设有n个独立样本对$(t_1, x_1), \cdots, (t_n, x_n)$，使用样本$t_i \sim p(t_i)$来估计加权函数的边缘概率密度函数$p(x)$，即

$$
\frac 1 n \sum_{i=1}^n \frac {p(t_i, x_i)} {\int_\tau p(t, x_i)\mathbf{d}t} \cdot \frac {f(x_i)} {p(t_i, x_i)} \approx \frac 1 n \sum_{i=1}^n \frac {p(t_i, x_i)} {\frac 1 n \sum_{j=1}^n \frac {p(t_j, x_i)} {p(t_j)}} \cdot \frac {f(x_i)} {p(t_i, x_i)}
$$

根据$p(t_i, x_i) = p(t_i) p(x_i | t_i)$，右边的近似可以简化为

$$
<I>_{SMIS} = \sum_{i=1}^n \frac {\dot{w}(t_i, x_i) f(x_i)} {p(x_i | t_i)} = \sum_{i=1}^n \frac {f(x_i)} {\sum_{j=1}^n p(x_i | t_j)}
$$

其中，$\dot{w}(t_i, x_i) = \frac {p(x|t_i)} {\sum_{j=1}^n p(x_i | t_j)}$。
由于在推导的过程中使用了近似，因此这个估计器是CMIS的有偏估计，但是，可以证明，这个估计器对积分I的估计仍然是无偏的，这个称为随机多重重要性采样(SMIS)估计器。

SMIS与n个采样技术每个技术一个样本的MIS是相似的，二者的区别在于SMIS的技术是从连续空间$\tau$中随机选择的。SMIS可以看作是将CMIS随机离散化后进行MIS.

SMIS中的权重在所有选择的采样技术上求和为一。而CMIS是在所有可用的采样技术上积分为一。因此，SMIS在有限子集上计算条件概率密度函数$p(x|t)$，而不是在计算边缘概率密度$p(x)$，因此SMIS提供了一个实际的方法来结合连续空间的采样技术。

## 方差分析
设，总计用$N$个采样技术-样本对，使用$SMIS_n$表示使用$N/n$个采样技术每个技术使用$n$个样本。使用$CMIS_u$来表示使用均匀加权函数的CMIS估计器，使用$CMIS_b$来表示使用平衡启发式加权方法加权函数的CMIS估计器。

实验表明，当$n$增加时，$SMIS_n$的方差逐渐接近$CMIS_b$，当$n=1$时，$SMIS_1$的方差等于$CMIS_u$。
然而，$SMIS_n$需要计算$n^2$个概率密度函数，随着$n$的增加，计算成本逐渐增加。


# 路径重用问题
上文中提到的路径重用问题可以使用连续多重重要性采样解决。

在路径重用问题中，我们可以将前缀$\bar{y_i}$看作采样技术而后缀$\bar{z}_i$看作时从这些技术中得到的样本，技术空间$\tau$时所有可能前缀的集合$\mathcal{P}$。

积分$I(\bar{y})$可以写为
$$
I(\bar{y}) = \int_\mathcal{P} \int_\tau w(\bar{y}', \bar{z}) \mathbf{d} \bar{y}' f_l(\bar{y}, \bar{z}) \mathbf{d} \bar{z} = \int_\mathcal{P} \int_\tau w(\bar{y}', \bar{z})  f_l(\bar{y}, \bar{z}) \mathbf{d} \bar{z} \mathbf{d} \bar{y}'
$$

其中，$w$是在空间$\tau$上正则化的。

对于n个样本，可以构建CMIS估计器，
$$
<I(\bar{y})>_{CMIS} = \frac 1 n \sum_{i=1}^n \frac {w(\bar{y}_i, \bar{z}_i) f(\bar{y}, \bar{z}_i)} {p(\bar{y}_i) p(\bar{z}_i | \bar{y}_i)}
$$

估计器通过将给定的前缀子路径$\bar{y}$与其他的后缀子路径$\bar{z}_i$连接起来构建完整的路径。
当使用最优得加权函数时，为了得到加权函数的值，需要计算为每个后缀子路径$\bar{z}_i$计算积分$p(z_i) = \int_\tau p(\bar{y}')p(\bar{z}_i | \bar{y}')\mathbf{d} \bar{y}'$的值，这个积分通常没有解析解，因此，可以使用SMIS来近似，即
$$
<I(\bar{y})>_{SMIS} = \sum_{i=1}^n \frac {\cdot{w}(\bar{y}_i, \bar{z}_i) f(\bar{y}, \bar{z}_i)} {p(\bar{z}_i | \bar{y}_i)} = \sum_{i=1}^n \frac {f(\bar{y}, \bar{z}_i)} {\sum_{j=1}^n p(\bar{z}_i | \bar{y}_j)}
$$

在上述公式中，$<I(\bar{y})>_{SMIS}$是无偏的估计器，它所有的项都是可计算的。

在实际的使用过程中，需要小心的选择参数$n$来控制重用路径的数量。每次连接一个后缀$\bar{z}_i$都需要计算$n$个概率密度函数$p(\bar{z}_i |\bar{y}_i)$，因此，每次估计都需要投射$n^2$条光线。此外，作者还假设在连接路径$\bar{y}$与$\bar{z}_i$和计算条件概率密度函数$p(\bar{z}_i | \bar{y}_j)$的时候，路径的两个端点总是可见的，这是一个合理的假设，因为每个$\bar{z}_i$都是从附近的$\bar{y}_i$构建的，对$\bar{y}_i$是可见的。
同时，还重用了$\bar{z}_i$的第一个节点的brdf，而不是在每次连接的时候都重新计算。
上述这些近似虽然引入了方差，但是，极大的减少了计算成本。

实验显示，连续多重重要性采样方法相较于之前的方法在路径重用中取得了较好的效果，可以显著的减少噪声。

# 总结
CMIS解决了无法将DMIS应用于连续空间的问题，并且证明了平衡启发式加权方法是最佳的。除此之外，本文还提供了无偏的SMIS近似，在实际中更加实用的技术组合框架，并且在路径重用、多波长采样和体积渲染中进行了应用，提升了渲染质量。

由于SMIS是对CMIS的近似，因此，平衡启发式加权方法不再是最优的方法，在这方面可以继续探索更优的加权方法。此外，由于减少方法的代价是增加计算成本，为了减少计算成本，在实际使用中也引入了几个假设，从而增加了方差，在设计新的加权方法时也可以将这方面考虑进去。

