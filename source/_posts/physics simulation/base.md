---
title: physics
date: 2022/02/26

---

# 数学概念

## 对角优势矩阵

指一矩阵的每一横行，对角线上的元素绝对值大于或等于同一横行的其他元素绝对值大小的和，即
$$\forall i, \ \ |a_{ii}| \ge \sum_{j \neq i} |a_{ij}|$$

# 物理概念

## stiffness(刚度)

刚度定义为施加的力与物体所产生的变形量的比值，表示材料或结构抵抗变形的能力，记为
$$ k = \frac P \delta$$
其中$k$表示刚度，$P$表示施力，$\delta$表示变形量（变形后的长度与原长度的差值）

### 与弹性模量的关系

弹性模量是材料组成的性质，是材料的内部性质；刚度是结构的性质，是固体的外延性质，取决与材料、形状和边界条件。
对于一个受压或受拉的元素，其轴向刚度为
$$k = \frac {AE} L $$
其中$A$为横截面积，$E$拉伸弹性模量，$L$为元素长度。
其转动刚度为
$$k = \frac {nEI} {L}$$
其中$I$为惯性矩，$n$是一个依赖于边界条件的整数（对于固端等于4）

# strain energy(应变能)

指物体在发生形变是贮存于其中的能量，当外力撤去后，应变能会逐渐释放。对于弹性材料，应变能可以完成全恢复，物体将回到初始状态，对塑性材料而言，应变能无法完全恢复。

应变能可表示为
$$U \int_v U_0 dV$$
其中$U_0$为应变能密度。
对一维线弹性材料
$$U_0 = \frac 1 2 \sigma \epsilon = \frac 1 2 E \epsilon^2 = \frac 1 2 \frac {\sigma^2} {\epsilon}$$
其中$\sigma$为应力，$\epsilon$为应变，$E$为杨氏模量

对于三维问题
$$U_0 = \frac 1 2 (\sigma_x \epsilon_x + \sigma_y \epsilon_y + \sigma_z \epsilon_z + \tau_{xy}\gamma_{xy} + \tau_{xz} \gamma_{xz} + \tau_{yz} \gamma_{yz} $$
其中$\sigma$为正应力，$\epsilon$为正应变，$\tau$和$\gamma$分别为切应力和切应变。

# Young's modulus(杨氏模量)

弹性模量。弹性材料承受正向应力时会产生正向应变，在型变量没有超过对应材料的一定限度时，定义正向应力与正向应变的比值为这种材料的杨氏模量，记为
$$E = \frac {\sigma} {\epsilon}$$
其中$\sigma$为正应力，$\epsilon$为正应变.

# 物理模拟方法

1. 根据力来模拟，计算在物体上施加的力，根据力计出加速度，在根据加速度计算出物体速度，再根据加速度计算出物体的新的位置。
2. 根据冲量来模拟，冲量可以计算得到速度，在再根据速度计算出物体的位置。
3. 基于位置的模拟方法，

# explicit eular

# semi-implicit eular

# Verlet integration(韦尔莱积分法)

数值稳定性比简单欧拉方法高很多，并保持了物理系统中的时间可逆性与相空间体积元体积守恒的性质。
根据牛顿运动方程
$$a(t) = \frac {f(t)} {m} \label{vi1}$$
粒子的坐标关于时间的方程为
$$s = r(t) \label{vi2}$$
将$ref{2}$进行泰勒展开并将$ref{vi1}$带入有
$$r(t + \Delta t) = r(t) + v(t)\Delta t + \frac {f(t)} {2m} \Delta t^2 + \frac 1 6 \frac {d^3 r} {dt^3} \Delta t^3 + O(\Delta t^4) \label{vi3}$$
同理有
$$r(t - \Delta t) = r(t) - v(t)\Delta t + \frac {f(t)} {2m} \Delta t^2 - \frac 1 6 \frac {d^3 r} {dt^3} \Delta t^3 + O(\Delta t^4) \label{vi4}$$
$ref{vi3}$和$ref{vi4}$相加有
$$r(t+\Delta t) + r(t - \Delta t) = 2 r(t) + \frac {f(t)} {m} \Delta t^2 + O(\Delta t^4) \label{vi5}$$
因此有
$$r(t + \Delta t) = 2r(t) - r(t - \Delta t) + \frac {f(t)} {m} \Delta t^2 + O(\Delta t^4) \label{vi6}$$
计算误差为时间的四阶。
由于公式$\ref{vi6}$中不存在速度，所以，要得到速度，则须从轨线中推到速度表达式
$$ v(t) = \frac {r(t+\Delta t) - r(t - \Delta t)} { 2 \Delta t} + O(\Delta t) \label{vi6}$$

## 速度表示的Verlet integration

更为常用，可以给出同一时间变量下的速度和位置，精度与基本verlet integration相同

首先，对方程$\ref{vi2}$进行泰勒展开
$$r(t + \Delta t) = r(t) + v(t) \Delta t + \frac {f(t)} {2m} \Delta t^2  \\
r(t + 2\Delta t) = r(t + \Delta t) + v(t + \Delta t) \Delta t + \frac {f(t + \Delta t)} {2m} \Delta t^2 $$
两式相减有
$$r(t + 2\Delta t) - r(t) = 2 r(t + \Delta t) + (v(t+\Delta t) - v(t))\Delta t + \frac {f(t + \Delta t) - f(t)} {2m} \Delta t^2$$
将基本verlet integration中的$t$换为$t+\Delta t$带入上式
$v(t+\Delta t) = v(t) + \frac{f(t + \Delta t) + f(t)} {2m} \Delta t$$

这里有点不清楚为什么是精度相同。

# Gauss-Seidel method(高斯-赛德尔迭代)

是数值线性代数中的一个迭代法，可用来线性方程组的近似解

对于一个含有$n$个未知量及$n$个等式的线性方程组
$$
a_{11} x_1 + \cdots + a_{1n} x_n = b_1, \\
a_{21} x_1 + \cdots + a_{2n} x_n = b_2, \\
\vdots
a_{n1} x_1 + \cdots + a_{nn} x_n = b_n, \\
$$
为了求解这个方程组的解$\bar{x}$，用$k$来计量迭代步数。给定一个方程组的近似值$\bar{x}^k \in \mathbb{R}^n$。在求解$k+1$不的近似值时，利用第$m$个方程组求解第$m$个未知量。在求解过程中，所有已解出的$k+1$步元素都被直接使用。
$$x_m^{k+1} = \frac 1 {a_{mm}} (b_m - \sum_{j=1}^{m-1} a_{mj} x_j^{k+1} - \sum_{j=m+1}^n a_{mj} x_j^k)$$

重复上述求解过程，可以得到一个线性方程组的近似值数列：$\bar{x}^0, \bar{x}^2, \cdots$, 若该数列收敛，则数列收敛于解$\bar{x}$

当该线性方程组系数矩阵为对称正定矩阵或对角优势矩阵时，数列收敛。

主对角线元素需要保证非零。

## 矩阵分解

线性方程组的系数矩阵$A$，可以分解为
$A = D + L + U$
其中$D$是一个对角矩阵，满足$(D)_{i,i} = (A)_{i,i}$，L为严格下三角矩阵，U为严格上三角矩阵。
高斯-塞尔德迭代的每一步可以写成如下形式
$$D\bar{x}^{k+1} = b - L\bar{x}^{k+1} - U\bar{x}^k$$
