---
title: math
date: 2022/03/10
---

#  Kronecker Delta(克罗内克$\delta$函数)
函数$\delta_{ij}$是一个二元函数，自变量是两个整数，若两者相等，则输出为1，否则为0.
$$
\delta_{ij} = \begin{cases}
1 & (i==j) \\
0 & (i \neq j)
\end{cases}
$$

kronecker函数具有筛选性，即对任意$j \in \mathbb{Z}$，有
$$
\sum_{i=-\infty}^\infty = \delta_{ij}a_i = a_j
$$
满足这个性质的函数称为具有Kronecker Delta性质。
