---
title: Finite Element method
date: 2022/03/04
---

# 有限元分析及应用

|       | 对象         | 特征         | 基本变量                          | 基本方程                         |
| ----- |:----------:|:----------:| ----------------------------- | ---------------------------- |
| 弹性力学  | 具有任意形状的变形体 | 小变形        | 变形方面的描述<br/>力的平衡描述<br/>材料物性描述 | 几何变形方程<br/>力的平衡方程<br/>物理本构方程 |
| 弹塑性力学 | 具有任意形状的变形体 | 变形（屈服，非线性） | 变形方面的描述<br/>力的平衡描述<br/>材料物性描述 | 几何变形方程<br/>力的平衡方程<br/>物理本构方程 |

变形体的力学要点

涉及三个方面：力的平衡、变形状态和材料行为

引入三大变量，三大方程

三大变量

+ 位移：$u(x)$

+ 应力：$\sigma(x) = F/A$

+ 应变：$\epsilon(x) = u / l$

三大力学方程

+ 平衡方程：$\sigma(x) = F/A$

+ 物理方程：$\sigma = E \epsilon$

+ 几何方程：$\epsilon(x) = u/l$

## 一维弹簧单元与杆单元

对于一个弹簧，采用基于节点的描述方式，如下图所示。

![](..\..\_img\physics_Figure_1.png "弹簧基于节点的描述")

劲度系数为k，左节点和右节点分别为1号节点和2号节点。1号节点的位移为$u_1$，2号节点的位移为$u_2$，1号节点受到的力为$F_1$，2号节点受到的力为$F_2$

根据上述表述，可以得到节点位移$(u_1, u_2)$和节点力$(F_1, F_2)$

因此，根据胡克定律有以下关系

$$
\begin{cases}
k(u_1 - u_2) = F_1 \\
k(u_2 - u_1) = F_2
\end{cases}
$$

上述关系可以写为矩阵形式，即

$$
\begin{bmatrix}
k & -k \\
-k & k
\end{bmatrix}
\begin{bmatrix}
u_1 \\ u_2
\end{bmatrix} = 
\begin{bmatrix}
F_1 \\ F_2
\end{bmatrix}
$$

上述关系称为弹簧的平衡方程或刚度方程。

将上述弹簧系统扩展为两个弹簧的系统

![](..\..\_img\physics_Figure_2.png "两弹簧系统")

两弹簧的劲度系数分别为$k_1$和$k_2$，长度分别为$l_1$和$l_2$，在节点$B$处有一个向右的外力$F_2$。

使用基于节点方法以上系统进行分析，如图所示

![1](..\..\_img\physics_Figure_3.png "两弹簧系统的节点描述")

可以得到弹簧1和弹簧2的平衡关系分别为

$$
\begin{cases}
k_1(u_1 - u_2) = F_{R1} \\
k_1(u_2 - u_1) = F_{I2}
\end{cases}
$$

$$
\begin{cases}
k_2(u_2 - u_3) = F_2 - F_{I2} \\
k_2(u_3 - u_2) = F_{R3}
\end{cases}
$$

整个系统的平衡方程为

$$
\begin{bmatrix}
k_1 & -k1 & 0 \\
-k_1 & k_1 + k_2 & -k_2 \\
0 & -k_2 & k_2
\end{bmatrix}
\begin{bmatrix}
u_1 \\
u_2 \\
u_3 
\end{bmatrix} = 
\begin{bmatrix}
F_{R1} \\
F_2 \\
F_{R3}
\end{bmatrix}
$$

对于杆单元而言，有物理关系

$$
\sigma = E \epsilon
$$

带入$\sigma = F / A$ 和$\epsilon = \delta / l$，整理有

$$
F = \frac {EA} {l} \delta
$$

因此，将弹簧系统中的劲度系数进行替换，就可以得到杆单元的平衡方程，即

$$
\frac {EA} {l} 
\begin{bmatrix}
1 & -1 \\
-1 & 1
\end{bmatrix}
\begin{bmatrix}
u_1 \\
u_2
\end{bmatrix} = 
\begin{bmatrix}
F_1 \\
F_2
\end{bmatrix}
$$

记，单元节点的位移为$q^e = \begin{bmatrix} u_1 & u_2\end{bmatrix}^T$，单元的节点力为$F^e = \begin{bmatrix} F_1 & F_2\end{bmatrix}^T$，单元的刚度矩阵为$K^e = \frac {EA} {l} \begin{bmatrix}1 & -1 \\ -1 & 1 \end{bmatrix}$。

因此，平衡方程可以简写为$K^e q^e = F^e$

## 杆单元的坐标变换

上一节中的公式是在杆的局部坐标系下定义的，在实际应用中，杆的变换信息是在世界坐标系下的，因此，需要杆单元进行坐标变换。

将局部坐标系$ox$变换为2维平面坐标系，局部坐标系的位移可以分解为两个方向上的分量，两坐标系的夹角为$\alpha$，如下图所示

![](..\..\_img\physics_Figure_4.png)

局部坐标系的节点位移为$q^e = \begin{bmatrix} u_1 & u_2 \end{bmatrix}^T$，世界坐标系的位移为$\bar{q}^e = \begin{bmatrix} \bar{u}_1 & \bar{v}_1 & \bar{u}_2 & \bar{v}_2 \end{bmatrix}$

两者之间的变换矩阵为

$$
\begin{bmatrix}
\cos \alpha & \sin \alpha & 0 & 0 \\
0 & 0 & \cos \alpha & \sin \alpha
\end{bmatrix}
$$

因此，有

$$
q^e = \begin{bmatrix} 
u_1 \\ u_2 
\end{bmatrix} = 
\begin{bmatrix}
\cos \alpha & \sin \alpha & 0 & 0 \\
0 & 0 & \cos \alpha & \sin \alpha
\end{bmatrix} 
\begin{bmatrix}
\bar{u}_1 \\ \bar{v}_1 \\ \bar{u}_2 \\ \bar{v}_2
\end{bmatrix} = T^e \cdot \bar{q}^e
$$

其中$T^e$称为坐标变换矩阵。带入原方程为

$$
K^e (T^e \bar{q}^e) = F^e \\
{T^e}^T K^e T^e \bar{q}^e = {T^e}^T F^e \\
\bar{K}^e \cdot \bar{q^e} = \bar{F}^e
$$

其中，$\bar{K}^e = {T^e}^T K^e T^e$是世界坐标系下的刚度矩阵，$\bar{F}^e = {T^e}^T F^e$是世界坐标系下的节点力矩阵。

对于空间杆单元，可以以类似的方法进行推到。其中，世界坐标系中的节点位移为$\bar{q}^e = \begin{bmatrix} \bar{u}_1 & \bar{v}_1 & \bar{w}_1 & \bar{u}_2 & \bar{v}_2 & \bar{w}_2 \end{bmatrix}^T$是一个6维的向量，刚度矩阵是一个$6\times6$的矩阵

## 指标记法

力在空间中表示，$F = \begin{bmatrix} F_1 & F_2 & F_3 \end{bmatrix} \ i=1,2,3$，其中$F_i$表示力在$x_i$轴上的分量。

几个基本概念

* 自由指标：每项中只出现一次的下标，即$\sigma_{ij}$。$i,j$都是自由指标，沿坐标轴自动变化

* 哑指标：每项中重复出现的下标，哑指标意味着求和，即$a_{ij}x_j = b_i$意味着$\sum_{j=1}^3 a_{ij}x_j = b_i$

* viogt标记：将高阶自由指标的张量携程低阶张量形式的过程叫做Voigt标记，其规则叫做Voigt移动规则，如二阶张量$\sigma_{ij} = \begin{bmatrix} \sigma_{11} & \sigma_{12} \\ \sigma_{21} & \sigma_{22}\end{bmatrix}$可以写成一维的向量，规则规定为先写对角线的分量，然后按逆时针写上三角部分，即$\sigma_{ij} = \begin{bmatrix} \sigma_{11} & \sigma_{22} & \sigma{12}\end{bmatrix}^T$

## 平面复杂变形体描述的建模思路

核心问题是如何处理复杂的几何形状

思路是将复杂的几何形状分为内部描述和外部描述。

几何体的内部使用函数来表达，给定坐标可以得到对应的值。为了进行分析，在物体内部取一个微小单元来对物体内部进行分析，平面问题采用微小面元$\mathbf{d}x\mathbf{d}y\_t$。

对于外部，对物体表面任意一点的边界条件进行建模，主要是位移约束的条件和受力的条件。

对于内部和外部，需要在微小单元上定义位移、应力和应变三大变量。基于这些变量构建平衡方程、物理方程和几何方程三大方程。

### 位移和几何方程

位移用$u(x,y)$和$v(x,y)$，$u(x,y)$表示点$(x,y)$沿着$x$轴的位移分量，$v(x,y)$表示点$(x,y)$沿着$y$轴的分量。

如图中的所示，矩形是变形前的物体，菱形是变形后的物体。

![](..\..\_img\physics_Figure_6.png)

平面变形量的定义需要考虑两个方面

1. 沿各个方向上的长度相对变化量

2. 夹角的变化量

$x$方向的相对伸长量为$\epsilon_{xx} = \frac {P'A' - PA} {PA}$

从图中可知，$P$的坐标为$(x,y)$，$P'$相对于$P$的位移量为$(u(x,y), v(x,y))$。A的坐标为$(x + \mathbf{d}x, y)$，$A$的水平位移量为$u(x + \mathbf{d}x, y)$，用泰勒级数展开后为$u(x,y) + \frac {\partial u(x,y)} {\partial x} \mathbf{d}x$。竖直位移量为$v(x + \mathbf{d}x, y)=v(x,y) + \frac {\partial v(x,y)}{\partial x} \mathbf{d}x$，同理，$B$点的竖直位移量为$v(x,y) + \frac{\partial v(x,y)} {\partial y} \mathbf{d}y$。

由于变形的角度很小，可以用水平的长度代替$P'A'$的长度，因此可知，$P'A'$的长度为$\mathbf{d}x  + \frac {\partial u} {\partial x} \mathbf{d}x$，$PA$的长度为$\mathbf{d}x$。

因此，$x$方向的相对伸长量为$\epsilon_{xx} = \frac {\partial u} {\partial x}$，同理，$y$方向的相对伸长量为$\epsilon_{yy} = \frac {\partial v} {\partial y}$。

$P'A'$和$PA$的夹角，使用点$A$的竖直位移量与$P$点的竖直位移量比上$PA$的原长求得，即

$$
\tan \alpha = \frac {(v + \frac {\partial v} {\partial x} \mathbf{d} x) - v} {\mathbf{d}x} = \frac {\partial v} {\partial x}
$$

由于角度比较小，可以用$\tan \alpha$近似$\alpha$角，即$\alpha = \frac {\partial v} {\partial x}$

同理可得，$P'B'$与$PB$的夹角为$\beta = \frac {\partial u} {\partial y}$

总夹角的变化定义为剪应变，即$\gamma_{xy} = \alpha + \beta = \frac {\partial v} {\partial x} + \frac {\partial u} {\partial y}$

综上所述，平面的几何方程为

$$
\begin{cases}
\epsilon_{xx} = \frac {\partial u} {\partial x} \\
\epsilon_{yy} = \frac {\partial v} {\partial y} \\
\gamma_{xy} = \frac {\partial v} {\partial x} + \frac {\partial u} {\partial y}
\end{cases}
$$

写成指标的形式为$\epsilon_{ij} = \frac 1 2 (u_{i,j} + u_{j,i})$

且约定$\epsilon_{ij} = \frac 1 2 \gamma_{ij}$

此外，这三个变形还应满足$\frac {\partial^2 \epsilon_{xx}} {\partial y^2} + \frac{\partial^2 \epsilon_{yy}} {\partial x^2} = \frac {\partial^2 \gamma_{xy}} {\partial x \partial y}$称为变形协调条件。

### 应力和平衡方程

平面问题需要对微小面元$\mathbf{d}x\mathbf{d}y\_t$进行受力分析获得应力信息。

在平面问题中，所有的力都在平面中变化。因此，每个微小面元有四个侧面受力，每个力都可以沿着两个方向进行分解，每个面有两个分力，共有八个分力。

分解后垂直于表面的力称为正应力，记为$\sigma_{xx}$和$\sigma_{yy}$，平行于表面的力称为剪应力，记为$\sigma_{xy}$和$\sigma_{yx}$。两个下标中第一个下标表示里的方向，第二个下标表示受力面法线的方向。

对于如下图所示的微小面元，厚度为$t$

![](..\..\_img\physics_Figure_5.png)

对于左边的侧面，坐标为$(x,y)$，因此，正应力为$\sigma_{xx}(x,y)$，剪应力为$\sigma_{yx}(x,y)$，对于右边的侧面，正应力和剪应力分别为$\sigma_{xx}(x + \mathbf{d}x, y)$和$\sigma_{yx}(x+\mathbf{d}x, y)$。

对于下表面正应力和剪应力分别为$\sigma_{yy}(x, y)$和$\sigma_{xy}(x, y)$。上表面正应力和剪应力分别为$\sigma_{yy}(x, y + \mathbf{d}y)$和$\sigma_{xy}(x, y + \mathbf{d}y)$。

若物体有体积力，则也用两个分量$\bar{b}_{x}$和$\bar{b}_{y}$来表示。

在构建平衡方程的时候，应考虑四个侧面的合理的平衡。

应力在经过$\mathbf{d}x$和$\mathbf{d}y$变化后的位置上有增量的表达。

因为取得是一个微小的面元，因此可以认为应力在各个侧面上是均匀分布的。

根据上述的信息，需要构建的平衡有三个

1. 沿$x$方向所有合力的平衡

2. 沿$y$方向所有合力的平衡

3. 所有合力关于任一点的力矩平衡

对于$x$方向上的合力有

$$
-\sigma_{xx}(x, y) \cdot \mathbf{d}y \cdot t + \sigma_{xx}(x + \mathbf{d}x, y) \cdot \mathbf{d}y \cdot t + \sigma_{xy}(x, y + \mathbf{d} y) \cdot \mathbf{d} x \cdot t - \sigma_{xy}(x,y) \cdot \mathbf{d}x \cdot t + \bar{b}_x \mathbf{d}x\mathbf{d}yt= 0
$$

使用泰勒展开并化简，最终结果为

$$
\frac {\partial \sigma_{xx}} {\partial x} + \frac {\partial \sigma_{xy}} {\partial y} + \bar{b}_x = 0
$$

其中，第一向表示正应力沿表面方向的变化，即两个左右两个平面受到的正应力的差值。第二项表示剪应力沿表面方向的变化，即剪应力沿表面变化的差值，$y$方向两个平面的剪应力为水平的，所以是对$y$求偏导，第三项是体积力的水平分量。

沿$y$方向的平衡关系同理可得

$$
\frac {\partial \sigma_{yy}} {\partial y} + \frac {\partial \sigma_{yx}} {\partial x} + \bar{b}_y = 0
$$

在计算力矩的平衡时，为了简单起见，取面元的中心位置来计算力矩的平衡。因为，正应力和体积力过中心，力臂为零，所以力矩计算中只有剪应力，得到的最终结果为

$$
\sigma_{xy} = \sigma_{yx}
$$

即，剪应力互等定理。

所以，最终的平衡方程为

$$
\begin{cases}
\frac {\partial \sigma_{xx}} {\partial x} + \frac {\partial \sigma_{xy}} {\partial y} + \bar{b}_x = 0 \\
\frac {\partial \sigma_{yy}} {\partial y} + \frac {\partial \sigma_{xy}} {\partial x} + \bar{b}_y = 0
\end{cases}
$$

写为指标的形式为$\sigma_{ij,j} + \bar{b}_i = 0$，其中下标$,j$表示物理量对$j$求偏导数。

### 物理方程

平面问题在拉伸时需要考虑两个方向的变化

1. 主方向上的伸长为$\epsilon_{xx} = \frac {\sigma_{xx}} {E}$

2. 垂直方向上的缩短为$\epsilon_{yy} = -\mu \epsilon_{xx}$

其中$\mu$时泊松比。

根据上述情况，可以给出平面上的胡克定理。

上节已知，每个平面有两个力，因此，可以将这些力分解成$x$方向的一组拉伸、$y$方向的一组拉伸以及四个剪应力的变化

对于$x$方向的力，会导致$x$方向拉伸和$y$方向缩短，因此有

$$
\begin{cases}
\epsilon_{xx} = \frac {\sigma_{xx}} {E} \\
\epsilon_{yy} = -\frac {\mu \sigma_{xx}} {E}
\end{cases}
$$

同理，对$y$方向上的力，会导致$y$方向的拉伸和$x$方向的缩短，有

$$
\begin{cases}
\epsilon_{xx} = -\frac {\mu \sigma_{yy}} {E} \\
\epsilon_{yy} = \frac {\sigma_{yy}} {E}
\end{cases}
$$

对于剪应力，只会导致角度的变化，而不会发生拉伸，因此，之后剪切角的变化。

从上文可知，角度的变化就是剪应变$\gamma_{xy}$，因此，剪应变和剪应力之间满足线性关系，即

$$
\gamma_{xy} = \frac {\sigma_{xy}} {G}
$$

其中，$G$是剪切模量。

将上述三组方程进行组合，可以得到平面上广义的胡克定律

$$
\begin{cases}
\epsilon_{xx} = \frac 1 E (\sigma_{xx} - \mu \sigma_{yy}) \\
\epsilon_{yy} = \frac 1 E (\sigma_{yy} - \mu \sigma_{xx}) \\
\gamma_{xy} = \frac 1 G \sigma_{xy}
\end{cases}
$$

其中，弹性模量$E$、剪切模量$G$和泊松比$\mu$只有两个是独立变量，因此，三者的关系有$G = \frac {E} {2(1+\mu)}$

将上述方程写成指标的形式为$\epsilon_{ij} = C_{ijkl} \sigma_{kl}$。其中$C_{ijkl}$是四阶张量，具体定义如下

|                  | $\sigma_{11}$ | $\sigma_{22}$ | $\sigma_{12}$ |
|:----------------:|:-------------:|:-------------:|:-------------:|
| $\epsilon_{11}$  | $1/E$         | $-\mu/E$      | 0             |
| $\epsilon_{22}$  | $-\mu / E$    | $1/E$         | 0             |
| $2\epsilon_{12}$ | 0             | 0             | $1/G$         |

力形式的方程为

$$
\begin{cases}
\sigma_{xx} = \frac E {1 - \mu^2} (\epsilon_{xx} + \mu \epsilon_{yy}) \\
\sigma_{yy} = \frac E {1 - \mu^2 }(\epsilon_{yy} + \mu \epsilon_{xx}) \\
\sigma_{xy} = G \gamma_{xy}
\end{cases}
$$

将上述方程写成指标的形式为$\sigma_{ij} = D_{ijkl} \epsilon_{kl}$。其中$D_{ijkl}$是四阶张量，具体定义如下

|               | $\epsilon_{11}$     | $\epsilon_{22}$     | $\epsilon_{12}$ |
|:-------------:|:-------------------:|:-------------------:|:---------------:|
| $\sigma_{11}$ | $E/(1-\mu^2)$       | $\mu E / (1-\mu^2)$ | 0               |
| $\sigma_{22}$ | $\mu E / (1-\mu^2)$ | $E/(1-\mu^2)$       | 0               |
| $\sigma_{12}$ | 0                   | 0                   | $G$             |

二阶张量之间需要通过四阶张量来表示。

通过Voigt规则，写出物理方程的矩阵形式

$$
\sigma_p = \bar{D}_{pq} \epsilon_q
$$

### 边界条件

位移边界$s_u$和力平衡的边界$s_p$，力的边界覆盖了除位移边界的其他地方，若不受力，则力为0。因此，一个物体G的表面由$s_u$和$s_p$组成。

在位移边界条件$s_u$处，物体的位移为给定的位移，即

$$
\begin{cases}
u = \bar{u} \\
v = \bar{v}
\end{cases}
$$

其中$\bar{u}$和$\bar{v}$是位移边界条件知道的位移。

在力的边界处，取出一个微元$\mathbf{d}x\mathbf{d}y_t$进行受力分析，考察其平衡问题，如下图所示

![](..\..\_img/physics_Figure_7.png)

对于$x$方向上的合力平衡有

$$
-\sigma_{xx} \cdot \mathbf{d} y \cdot t - \sigma_{xy} \cdot \mathbf{d}x\cdot t + \bar{p}_x \cdot \mathbf{d} s \cdot = 0
$$

化简可得

$$
\sigma_{xx} \cdot n_x + \sigma_{xy} n_y = \bar{p}_x
$$

其中，$n_s = \mathbf{d}y / \mathbf{d}s$和$n_y = \mathbf{d}x /\mathbf{d}s$为外表面法线$n$的方向余弦

同理，$y$方向上有

$$
\sigma_{yy} \cdot n_y + \sigma_{yx} n_x = \bar{p}_y
$$

再加上剪应力互等定律

写成指标形式为$\sigma_{ij}n_j =\bar{p}_i$

### 平面刚体位移

$$
\begin{cases}
u(x,y) = -\omega_0 y + u_0 \\
v(x,y) = -\omega_0 x + v_0
\end{cases}
$$

其中，$u_0,v_0,\omega_0$表征刚体位移的常数

$u_0$表示整个物体在$x$方向的刚体平移量

$v_0$表示整个物体在$y$方向的刚体平移量

$\omega_0$为整个刚体转动的角度

已知$u$和$v$可以得到刚体的转动量，即$\omega_0 = \frac 1 2 (\frac {\partial v} {\partial x} - \frac {\partial u} {\partial y})$

## 空间问题描述

3D问题可以从2D问题进行拓展

因此，3D问题的三大基本变量为

1. 位移分量为$u(x,y,z),v(x,y,z),w(x,y,z)$

2. 应变分量为$\epsilon_{xx}(x,y,z),\epsilon_{yy}(x,y,z),\epsilon_{zz}(x,y,z),\gamma_{yz}(x,y,z),\gamma_{xz}(x,y,z),\gamma_{xy}(x,y,z)$

3. 应力分量为$\sigma_{xx}(x,y,z),\sigma_{yy}(x,y,z),\sigma_{zz}(x,y,z),\sigma_{yz}(x,y,z),\sigma_{xz}(x,y,z),\sigma_{xy}(x,y,z)$

由于存在三组剪应力互等和三组剪应变互等，因此，独立应变分量和位移分量有6个

平衡方程为

$$
\begin{cases}
\frac {\partial \sigma_{xx}} {\partial x} + \frac {\partial \sigma_{xy}} {\partial y} + \frac {\partial \sigma_{xz}} {\partial z} + \bar{b}_x = 0\\
\frac {\partial \sigma_{xy}} {\partial x} + \frac {\partial \sigma_{yy}} {\partial y} + \frac {\partial \sigma_{yz}} {\partial z} + \bar{b}_y = 0\\
\frac {\partial \sigma_{xz}} {\partial x} + \frac {\partial \sigma_{yz}} {\partial y} + \frac {\partial \sigma_{zz}} {\partial z} + \bar{b}_z  = 0
\end{cases}
$$

几何方程为

$$
\begin{cases}
\epsilon_{xx} = \frac {\partial u} {\partial x} \\
\epsilon_{yy} = \frac {\partial v} {\partial y} \\
\epsilon_{zz} = \frac {\partial w} {\partial z} \\
\gamma_{xy} = \frac {\partial v} {\partial x} + \frac {\partial u} {\partial y}\\
\gamma_{yz} = \frac {\partial w} {\partial y} + \frac {\partial v} {\partial z}\\
\gamma_{zx} = \frac {\partial u} {\partial z} + \frac {\partial w} {\partial x}\\
\end{cases}
$$

物理方程为

$$
\begin{cases}
\epsilon_{xx} = \frac 1 E [\sigma_{xx} - \mu(\sigma_{yy} + \sigma_{zz})] \\
\epsilon_{yy} = \frac 1 E [\sigma_{yy} - \mu(\sigma_{zz} + \sigma_{xx})] \\
\epsilon_{zz} = \frac 1 E [\sigma_{zz} - \mu(\sigma_{xx} + \sigma_{yy})] \\
\gamma_{xy} = \frac 1 G \sigma_{xy}  \\
\gamma_{yz} = \frac 1 G \sigma_{zy}  \\
\gamma_{zx} = \frac 1 G \sigma_{zx}  \\
\end{cases}
$$

物理方程的逆形式为

$$
\begin{cases}
\sigma_{xx} = \frac {E(1-\mu)} {(1+\mu) (1-2\mu)} [\epsilon_{xx} + \frac {\mu} {1 - \mu}(\epsilon_{yy} + \epsilon_{zz})] \\
\sigma_{yy} = \frac {E(1-\mu)} {(1+\mu) (1-2\mu)} [\epsilon_{yy} + \frac {\mu} {1 - \mu}(\epsilon_{zz} + \epsilon_{xx})] \\
\sigma_{zz} = \frac {E(1-\mu)} {(1+\mu) (1-2\mu)} [\epsilon_{zz} + \frac {\mu} {1 - \mu}(\epsilon_{xx} + \epsilon_{yy})] \\
\sigma_{xy} = G\gamma_{xy} \\
\sigma_{yz} = G\gamma_{yz} \\
\sigma_{zx} = G\gamma_{zx}
\end{cases}
$$

几何边界条件为

$$
\begin{cases}
u(x,y,z)|_{x=x_0, y=y_0,z=z_0} = \bar{u} \\
v(x,y,z)|_{x=x_0, y=y_0,z=z_0} = \bar{v} \\
w(x,y,z)|_{x=x_0, y=y_0,z=z_0} = \bar{w} \\
\end{cases}
$$

外力边界条件为

$$
n_x \sigma_{xx}(x_0, y_0, z_0) + n_y \sigma_{xy}(x_0, y_0, z_0) + n_z \sigma_{xz}(x_0,y_0, z_0) = \bar{p}_x \\
n_x \sigma_{xy}(x_0, y_0, z_0) + n_y \sigma_{yy}(x_0, y_0, z_0) + n_z \sigma_{yz}(x_0,y_0, z_0) = \bar{p}_x \\
n_x \sigma_{xz}(x_0, y_0, z_0) + n_y \sigma_{yz}(x_0, y_0, z_0) + n_z \sigma_{zz}(x_0,y_0, z_0) = \bar{p}_x \\
$$

## 有限元的应用

### 平面问题的三节点三角形单元描述

对于平面三角形单元，三个节点的坐标位置为$(x_i, y_i)$，各个节点沿$x$方向和$y$方向的位移为$(u_i,v_i)$，得到位移矩阵$q = \begin{bmatrix} u_1 & v_1 & u_2 & v_2 & u_3 & v_3\end{bmatrix}^T$。

由于有三个节点，我们假设平面位移场函数为
$$
\begin{cases}
u(x, y) = \bar{a}_0 + \bar{a}_1 x + \bar{a}_2 y \\
v(x, y) = \bar{b}_0 + \bar{b}_1 x + \bar{b}_2 y
\end{cases}
$$

这个平面位移场函数应该满足以下条件
$$
\begin{cases}
u(x_i, y_i) = u_i \\
v(x_i, y_i) = v_i
\end{cases}
$$
即在节点出的函数值应该等于节点的位移值。

根据上述条件，可以得到线性方程组，六个方程，六个未知量。

解得待定系数为
$$
\begin{cases}
\bar{a}_0 = (a_1 u_1 + a_2 u_2 + a_3 u_3) / 2A \\
\bar{a}_1 = (b_1 u_1 + b_2 u_2 + b_3 u_3) / 2A \\
\bar{a}_2 = (c_1 u_1 + c_2 u_2 + c_3 u_3) / 2A \\
\bar{b}_0 = (a_1 v_1 + a_2 v_2 + a_3 v_3) / 2A \\
\bar{b}_1 = (b_1 v_1 + b_2 v_2 + b_3 v_3) / 2A \\
\bar{b}_2 = (c_1 v_1 + c_2 v_2 + c_3 v_3) / 2A \\
\end{cases}
$$
其中
$$
\begin{cases}
a_1 = x_2 y_3 - x_3 y_2 \\
a_2 = x_3 y_1 - x_1 y_3 \\
a_3 = x_1 y_2 - x_2 y_1 \\
b_1 = y_2 - y_3 \\
b_2 = y_3 - y_1 \\
b_3 = y_1 - y_2 \\
c_1 = -x_2 + x_3 \\
c_2 = -x_3 + x_1 \\
c_3 = -x_1 + x_2
\end{cases}
$$
$A$是三角形的面积。

将参数带回，将位移场函数用节点位移的形式表示，有
$$
\begin{cases}
u(x, y) = N_1(x,y) u_1 + N_2(x, y) u_2 + N_3(x,y) u_3 \\
v(x, y) = N_1(x,y) v_1 + N_2(x, y) v_2 + N_3(x,y) v_3
\end{cases}
$$
其中$N_i = (a_i + b_i x + c_i y) / 2A$

写成矩阵的形式有
$$
\textbf{u}(x,y) = 
\begin{bmatrix} u(x,y) \\ v(x,y) \end{bmatrix} = 
\begin{bmatrix}
N_1 & 0 & N_2 & 0 & N_3 & 0 \\
0 & N_1 & 0 & N_2 & 0 & N_3
\end{bmatrix} \cdot
\begin{bmatrix}
u_1 \\ v_1 \\ u_2 \\ v_2 \\ u_3 \\ v_3 
\end{bmatrix} = \textbf{N}(x,y) \textbf{q}^e
$$
其中，$\textbf{N}(x,y)$是形状矩阵函数。

得到平面位移场函数后，根据几何方程，可以得到应变场函数，即
$$
\epsilon (x,y) = 
\begin{bmatrix} \epsilon_{xx} \\ \epsilon_{yy} \\ \gamma_{xy} \end{bmatrix} =
\begin{bmatrix}
\frac {\partial u} {\partial x} \\
\frac {\partial v} {\partial y} \\
\frac {\partial u} {\partial y} + \frac {\partial v} {\partial x} \\
\end{bmatrix} = 
\begin{bmatrix} 
\frac {\partial} {\partial x} & 0 \\
0 & \frac {\partial} {\partial y} \\
\frac {\partial} {\partial y} & \frac {\partial} {\partial x}
\end{bmatrix} \cdot 
\begin{bmatrix} u(x,y) & v(x,y) \end{bmatrix} = [\partial ] \textbf{u}
$$
其中，$[\partial ]$为几何方程的算子矩阵。

将位移场$u$带入得，$\epsilon(x,y) = [\partial ] N(x,y) q^3 = B(x,y) q^e$

其中，$B(x,y)$是几何函数，为
$$
B(x,y) = \frac 1 {2A} 
\begin{bmatrix}
b_1 & 0 & b_2 & 0 & b_3 & 0 \\
0 & c_1 & 0 & c_2 & 0 & c_3 \\
c_1 & b_1 & c_2 & b_2 & c_3 & b_3
\end{bmatrix} = 
\begin{bmatrix}
B_1 & B_2 & B_3
\end{bmatrix}
$$

得到应变场后，根据物理方程，可以得到应力场的表达式
$$
\sigma(x,y) = \begin{bmatrix} \sigma_{xx} \\ \sigma_{yy} \\ \tau_{xy} \end{bmatrix} = 
\frac E {1 - \mu^2} \begin{bmatrix}
1 & \mu & 0 \\
\mu & 1 & 0 \\
0 & 0 & \frac {1-\mu} 2
\end{bmatrix} \cdot 
\begin{bmatrix} \epsilon_{xx} \\ \epsilon_{yy} \\ \gamma_{xy} \end{bmatrix} = D \cdot \epsilon
$$
其中，$D$是弹性系数矩阵为

若是从应力开始推到为应变的问题，$D$中的系数$(E, \mu)$应换为$(\frac {E} {1-\mu^2}, \frac {\mu} {1-\mu})$

带入应变场后得到
$$
\sigma = D \cdot B \cdot q^e = S \cdot q^e
$$
其中，$S$为应力函数矩阵

变形物体总势能为$\Pi = \Lambda - W$，即总应变能$\Lambda$与外力做功$W$的差。
在得到位移场后，外力做功很好计算，即$W = \int_\Omega \bar{b}^T u \mathbf{d} \Omega + \int_{S_p} \bar{p}^T u \mathbf{d} A$

应变能的计算为$\Lambda = \frac 1 2 \int_\Omega \sigma^T \epsilon \mathbf{d} \Omega$

因此，将前面计算的应变、应力带入并化简后有
$$
\Pi = \frac 1 2 {q^e}^T K^e q^e - {P^e}^T q^e
$$
其中$K^e$是刚度矩阵，即$K^e = \int_A B^T D B \cdot \mathbf{d} A \cdot t$，$t$是平面问题的厚度。

从上面可知，$B$矩阵是常系数矩阵，因此可以进行化简
$$
K^e = B^T D B tA = \begin{bmatrix}
k_{11} & k_{12} & k_{13} \\
k_{21} & k_{22} & k_{23} \\
k_{31} & k_{32} & k_{33}
\end{bmatrix}
$$
其中有
$$
k_{rs} = B^T_r D B_s t A = \frac {Et} {4 (1-\mu^2) A} 
\begin{bmatrix}
k_1 & k_3 \\
k_2 & k_4
\end{bmatrix}
$$
其中
$$
k_1 = b_r b_s + \frac {1 - \mu} {2} c_r c_s \\
k_2 = \mu c_r b_s + \frac {1 - \mu} {2} b_r c_s \\
k_3 = \mu b_r c_s + \frac {1 - \mu} {2} c_r b_s \\
k_4 = c_r c_s + \frac {1 - \mu} {2} b_r b_s
$$

$P^e$是单元节点等效载荷，即
$$
P^e = \int_A N^T \bar{b} t \mathbf{d} A + \int_l N^T \bar{p} t \mathbf{d} l
$$
其中，$l_p$是作用有外载荷的边，$\int \mathbf{d}l$为线积分。