---
title: position based dynamics
date: 2022/02/26
---

可以避免与显示积分有关的overshotting和能量增加问题。
消除不稳定问题
容易理解和实现

算法基本思路
1. 根据力计算出速度，在计算的速度上加上damping
2. 根据速度计算出粒子的新位置
3. 根据新的位置信息和旧的位置信息生成碰撞约束条件
4. 迭代约束条件，更新粒子位置。
5. 根据最终的位置信息，计算出速度。

一共有两种约束，一种是内部约束，在这个过程中不变的约束。另一个种是碰撞约束，即第三步中生成的约束，每次迭代都生成新的碰撞约束。

对于内部约束，约束函数C是旋转和变换，独立于刚体模式（不发生任何明显内部变形的平移或旋转）。所以$\nabla_p C$是垂直与刚体模式的。当修正的$\Delta p$沿$\nabla_p C$方向选择的，当所有点的质量相同时，两个动量会守恒。
因此，给定$p$需要找到一个正确的$\Delta p$使用的满足约束条件$C(p + \Delta p) = 0$，根据泰勒展开，可以近似为
$$C(p + \Delta p) \approx C(p) + \nabla_pC(p) \Delta p = 0$$
由于限制了$\Delta p$沿$\nabla_p C$方向，因此有
$$\delta p = \lambda \nabla_p C(p)$$
两个方程联立可以解得$\lambda$，因此可以解得
$$\Delta p = - \frac {C(p)} {\nabla_p C(p)} \nabla_pC(p)$$

因此，对于任何一个点$p_i$和约束$(p_1,\cdots, p_n)$，有
$$\Delta p_i = -s \nabla_{p_i} C(p_1, \cdots, p_n)$$
其中$s$为
$$s = \frac {C(p_1, \cdots, p_n)} {\sum_j |\nabla_{p_j} C(p_1, \cdots, p_n)}$$

当节点的质量不同的时候，通过增加权重$w_i =1/m_i$来进行修正
此时的结果为
$$\Delta_{p_i} = - \frac {n \cdot w_i} {\sum_j w_j} \nabla_{p_i}C(p1, \cdots, p_n)$$

在第四步中，移动点的位置需要满足几个约束条件。
1. 线性动量守恒，$\sum_i m_i \Delta p_i = 0$。相当于保证质心不变。
2. 角动量守恒，$\sum_i r_i \times m_i \Delta p_i = 0$，其中$r_i$是$p_i$到任意共同旋转中心的距离
