---
title: physics
date: 2022/02/26
---

# Lagrangian Mesh Based Methods
## The Finite Element Method
动态弹性物体的偏微分方程为
$$\rho \ddot{x} = \nabla \sigma + f$$
其中，$\rho$是材料密度，$f$是外力，$\nabla \sigma$是应力，密度可以看作质量对位置的导数，因此，偏微分方程可以看作是牛顿第二定律$F=ma$的对位置的微分形式。

FEM将连续物体离散为一组基本的相互连接的单元，在这组离散单元上求解微分方程，从而近似求解整体方程。


使用了显示有限元的论文有[OH99](#OH99)。

在显示有限元中，质量、内力和外力都位于顶点。网格中的顶点都被视为质点，每个榕树就像一个连接所有相邻质点的广义弹簧。

给定元素节点位置和固定的基函数，可以计算出元素中的连续的变形场，根据变形场可以计算出应变场和应力场，根据元素的变形能量公式。力则是能量关于位置的倒数。当节点应力和节点位置是线性关系的时候（通常是非线性的），两者的关系为
$$
f_e = K_e u_e
$$
其中$K_e$是元素的刚度矩阵。
此时，运动的线性方程为
$$
M \ddot{u} + D \dot{u} + Ku = f_{ext}
$$
其中$M$是质量矩阵，$D$是阻尼矩阵，$f_{ext}$是应用在物体上的外力。通常，$M$和$D$的对角矩阵。

单材料是各向异性的时候，PDE为
$$
\rho \ddot{xx} = \mu \Delta u + (\lambda + \mu)(\nabla)(\nabal \cdot u)
$$
其中$\lambda$和$\mu$可以从杨氏模量和泊松比来计算。

方程在[DDBC99](#DDBC99)中得到解决。

基于离散高斯散度理论，[DDCB00](#DDCB00)进一步改进了离散算子，得到了更加准确的结果。

[OH99](#OH99)和[OBH02](#OBH02)中提出了一种基于有限元的技术，用于模拟与弹塑性材料有关的脆性和人形断裂。
其他关于脆性断裂的可视化模拟的其他研究有[SWB00](#SWB00)和[MMDJ01](#MMDJ01)

线性弹性力尽在小变形的情况下有效，大的旋转变形会产生不准确的恢复力。 [MG04]中提取了旋转部分，然后计算非旋转相关的力。此时公式可以变为
$$
f = RK(R^Tx - x_0)
$$
其中，$R$是一个$12 \times 12$的矩阵，在对角线上是4个相同的$3 \times 3$的旋转矩阵。这可以产生稳定快速的结果。
[CGC*02](#CGC*02)提出了另一个解决方法，这里不详述了。
[THMG04](#THMG04)中描述了三角形和四面体网格的距离、表面和体积的保存。

## The Method of Finite Differences


## The Finite Volume Method
在显示有限元方法中，力是通过求势能对节点位置的数据计算的。而在有限体积方法中，有更直接的方法可以得到力，即应力张量可以用来计算相对于某一平面方向的单位面积内力f，
$$
f = \sigma n
$$
$n$是平面的单位法向量。因此，施加在有限元平面A的力可以通过面积分得到，即
$$
f_a = \int_A \sigma \mathbf{d}A
$$
当使用线性基函数时(即单个单元时三角形或四面体时)，应力时常量，此时，力可以简化为乘积，即
$$
f_a = A \sigma n
$$
为了得到节点的力，循环所有的面元，并将面元上的力平均分配的节点上。

[THBF03](#TBHF03)中使用了这个方法。

## The Boundary Element Method
边界元法所有的计算都在物体表面。通过格林高斯公式将体积分转换为面积分。将积分维度从三维降到二维，从而加速了计算。

缺点在于只适用于物体内部时同质的材料，且拓扑结构的变化难以处理。

[JP99](#JP99)、[JP02b](#JP02b)和[JP03](#JP03)中使用了这个方法。

# Mass-Spring System
这个弹簧系统可以用牛顿第二定律来表示
$$
M \ddot{x} = f(x,y)
$$
其中$M$时质量矩阵。
弹簧质点系统仅需解一个耦合的常微分方程，可以使用数值积分的方法求解。

各向异性材料中的弹簧属性还与朝向有关。

弹簧通常被建模为线性的弹性系统，因此，在质点上的力有连接之间的弹簧确定，即
$$
f_i = k_s (|x_{ij} - l_{ij}) \frac {x_{ij}} {|x_{ij}|}
$$
其中$x_{ij}$时弹簧相连的两个质点位置向量$x_j - x_i$，$k_s$时弹簧的刚度，$l_{ij}$是弹簧的原长。

由于存在能量耗散，因此，每个弹簧被施加一个阻尼。通常为
$$
f_i = k_d \frac {v_{ij}^T x_{ij}} {x_{ij}^Tx_{ij}} x_{ij}
$$
其中，速度$v_{ij} = v_j - v_i$。阻尼的大小与速度在位移方向上的投影有关。




# Reference
1. <span id="OH99">Graphical modeling and animation of brittle fracture</span>
2. <span id="DDBC99">Interactive multiresolution animation of deformable models</span>
3. <span id="DDBC00">Adaptive simulation of soft bodies in real-time</span>
4. <span id="OBH02">Graphical modeling and animation of ductile fracture</span>
5. <span id="SWB00">Fast and controllable simulation of the shattering of brittle objects</span>
6. <span id="CGC*02">Real-time simulation of deformation and fracture of stiff materials</span>
7. <span id="MMDJ01">Interactive skeleton-driven dynamic deformation</span>
8. <span id="THMG04">A versatile and robustmodel forgeometricallycomplex deformablesolids</span>
9. <span id="THBF03">Finite volume methods forthe simulationofskeletal muscle</span>
10. <span id="JP99">ArtDefo: accurate real time deformable objects.</span>
11. <span id="JP02b">Real time simulation of multizone elastokinematic models</span>
12. <span id="JP03"> Multiresolution Green’s function methods for interactive simulation of large-scale elastostatic objects</span>