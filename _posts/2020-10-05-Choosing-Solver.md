---
layout: post
mathjax: true
title: Choosing the correct solver type - Dense or Sparse
date: 2020-10-05
category:
  - Blog
tags:
  - Python
  - Code
  - Optimization
  - Performance
---

The prevailing opinion for solving mathematical programming problems is to use sparse solvers that utilize structure and sparsity to accelerate solution speeds. However, this is not always advantageous. For example, when solving smaller problems will little structure or a relatively small number of nonzero's in the constraint matrix, dense solvers can be radically faster.

To see an approximate crossover to where it would make sense to transferring over to a sparse solver. I have made a quick example problem defined as follows.

$$\min_{x}$$
$$\text{s.t:}\mathcal{I}_{n}x\leq \mathcal{1}$$
$$\text{    }-\mathcal{I}_{n}x\leq \mathcal{1}$$

This problem is a prime candidate for sparse optimization solvers when the dimension becomes large, but what about when the dimension is small? This is the average for 100 runs at each data point. Solution times are in milliseconds, and N is the number of dimensions being considered.

|    n    |  GLPK  |  Gurobi  | Speed Factor |
|:-------:|:------:|:--------:|:------------:|
|    10   |  .043  |   1.04   |     24.2     |
|    50   |   .16  |    1.5   |      9.4     |
|   100   |   .49  |    2.3   |      4.7     |
|   500   |  10.5  |   11.1   |     1.05     |
|   1000  |   44   |   30.3   |      .69     |
|   2000  |   188  |    94    |      .5      |

So this type of problem gives a larges advantage to sparse solvers, and the sparse solver is only faster than the dense solver at approximately 500 dimensions. This was surprising to me, so I figured I would pass the information on. The same behavior can be seen even with activating random constraints.
