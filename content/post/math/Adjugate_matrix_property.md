---
date: 2020-06-08
title: 伴随矩阵恒等式的证明
tags: ['Math']
categories: ['Math']
---

伴随矩阵最基本的恒等式为:
$$ A(A^*) = |A|E $$

怎么证明呢? 参考了[这里](https://byjus.com/jee/adjoint-and-inverse-of-a-matrix/)

直接从定义出发来证明。
以一个 3x3 的具体行列式为例。

$$
A = \begin{vmatrix}
  a_{11} & a_{12} & a_{13} \\\\
  a_{21} & a_{22} & a_{23} \\\\
  a_{31} & a_{32} & a_{33} \\\\
\end{vmatrix}
$$

根据定义，A 的伴随矩阵 A^\*为每个元素的代数余子数按列排组成的矩阵。

$$
A^* = \begin{vmatrix}
  A_{11} & A_{21} & A_{31} \\\\
  A_{12} & A_{22} & A_{32} \\\\
  A_{13} & A_{23} & A_{33} \\\\
\end{vmatrix}
$$

所以:

$$
A * A^* = \begin{vmatrix}
  a_{11} & a_{12} & a_{13} \\\\
  a_{21} & a_{22} & a_{23} \\\\
  a_{31} & a_{32} & a_{33} \\\\
\end{vmatrix} \begin{vmatrix}
  A_{11} & A_{21} & A_{31} \\\\
  A_{12} & A_{22} & A_{32} \\\\
  A_{13} & A_{23} & A_{33} \\\\
\end{vmatrix} = \\\\
\begin{vmatrix}
  a_{11}A_{11} + a_{12}A_{12} + a_{13}A_{13} & a_{11}A_{21} + a_{12}A_{22} + a_{13}A_{23} & a_{11}A_{31} + a_{12}A_{32} + a_{13}A_{33} \\\\
  a_{21}A_{11} + a_{22}A_{12} + a_{23}A_{13} & a_{21}A_{21} + a_{22}A_{22} + a_{23}A_{23} & a_{21}A_{31} + a_{22}A_{32} + a_{23}A_{33} \\\\
  a_{31}A_{11} + a_{32}A_{12} + a_{33}A_{13} & a_{31}A_{21} + a_{32}A_{22} + a_{33}A_{23} & a_{31}A_{31} + a_{32}A_{32} + a_{33}A_{33} \\\\
\end{vmatrix} = \\\\
\begin{vmatrix}
  |A| & 0 & 0 \\\\
  0 & |A| & 0 \\\\
  0 & 0 & |A| \\\\
\end{vmatrix} = |A|E
$$
