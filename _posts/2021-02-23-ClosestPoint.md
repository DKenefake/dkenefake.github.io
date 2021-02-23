---
layout: post
mathjax: true
title: Closest point between two convex polytopes 
date: 2021-2-23
category:
  - Blog
tags:
  - Optimization
  - Math
  - Geometry
---

I have been doing some geometry calculations for a project recently, and I solved a problem in a way that I thought was pretty neat. The problem is calculating the closest pair of points between two convex polytopes. Surprisingly this is a fairly common calculation that crops up in optimal control applications. To get some definitions out of the way, we are defining an $n$ dimensional polytope as $\mathcal{P} = \{x\in\mathcal{R}^n : Ax\leq b\}$. Where a two-dimensional polytope is your regular polygon, and the simplest 2-dimensional polygon is a triangle. The problem we are trying to solve is the following optimization problem.

$$
\begin{align*}
\min\quad|| x_1 - x_2||_1\\
  A_1 x_1 &\leq b_1\\
  A_2 x_2 &\leq b_2\\
  x_1, x_2&\in\mathcal{R}^n
\end{align*}
$$

Here, if the above problem's optimal value is zero, we can see that these polytopes must share a point. In other words, if the minimum distance between the two points in different polytopes is zero, then these points must be the same point. The problem, as stated, is not easy (due to the $l1$ norm) in the objective function, but this is fixable by reformulating the above optimization problem with a minimax type approach. By introducing an additional variable and constraint, we can reformulate this problem into an equivalent linear program readily solvable. Here the added variable $t$ is the bounds on the difference of the two points.

$$
\begin{align*}
\min \quad& t\\
  \text{s.t. }  A_1 x_1 &\leq b_1\\
  A_2 x_2 &\leq b_2\\
  -t \leq x_{1} -& x_{2} \leq t\\
  x_1, x_2&\in\mathcal{R}^n\\
  t&\in\mathcal{R}
\end{align*}
$$

As an interesting note, we can solve the $l\infty$ problem by changing what constraints we are adding. If we change the added constraints to each dimension of the points, we are trying to minimize each dimension's largest difference.

$$
\begin{align*}
\min \quad& t\\
  \text{s.t. }  A_1 x_1 &\leq b_1\\
  A_2 x_2 &\leq b_2\\
  -t \leq x_{1,j} -& x_{2,j} \leq t, \forall j \in\{1,\dots, n\}\\
  x_1, x_2&\in\mathcal{R}^n\\
  t&\in\mathcal{R}
\end{align*}
$$

This solution method is not necessarily the fastest method to solve this problem, as there are specialized algorithms for special cases and low dimensions.  
