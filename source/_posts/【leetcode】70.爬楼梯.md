---
title: '[leetcode]70.爬楼梯'
date: 2024-05-05 15:12:36
tags:
  - leetcode
---

假设你正在爬楼梯。需要 n 阶你才能到达楼顶。

每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？

示例 1：
```
输入：n = 2
输出：2
解释：有两种方法可以爬到楼顶。
1. 1 阶 + 1 阶
2. 2 阶
```

示例 2：
```
输入：n = 3
输出：3
解释：有三种方法可以爬到楼顶。
1. 1 阶 + 1 阶 + 1 阶
2. 1 阶 + 2 阶
3. 2 阶 + 1 阶
```

提示：
```
1 <= n <= 45
```


# 解法一：动态规划


# 解法二：矩阵快速幂

$$
\begin{bmatrix}
{1}&{1}\\
{1}&{0}\\
\end{bmatrix}
\begin{bmatrix}
{f(n)}\\
{f(n-1)}\\
\end{bmatrix}=
\begin{bmatrix}
{f(n)}{+}{f(n-1)}\\
{f(n)}\\
\end{bmatrix}=
\begin{bmatrix}
{f(n+1)}\\
{f(n)}\\
\end{bmatrix}
$$

$$
\begin{bmatrix}
{f(n+1)}\\
{f(n)}\\
\end{bmatrix}=
\begin{bmatrix}
{1}&{1}\\
{1}&{0}\\
\end{bmatrix}^{n}
\begin{bmatrix}
{f(1)}\\
{f(0)}\\
\end{bmatrix}
$$


