---
layout: post
mathjax: true
title: Constrained Min Norm - Linear Equality Constraints
date: 2020-08-25
category:
  - Blog
tags:
  - Python
  - Code
  - Math
  - Optimization
  - Regression
---
Recently I have been reading on how to regress with constraints. It has uses in fitting models to data as well as signal reconstruction. In this post we will look at how to solve optimization problems of the following form.

$$\min_{x} ||x||^2, \text{ s.t.: } Cx = d$$

The Lagrange function of this problem after scaling is the following

$$L(x,\lambda) = \frac{1}{2}x^T\mathcal{I}x + \sum_{i=0}^n\lambda_i(C_ix-d_i)$$

The Optimality Conditions are

$$\frac{\partial \mathcal{L}}{\partial x_j}(x^{*}, \lambda) = 0, j\in\{0,\dots m\} $$

$$\frac{\partial \mathcal{L}}{\partial \lambda_i}(x^{*}, \lambda) = 0,i\in\{0,\dots n\}$$

By evaluating the partial derivatives, we can see the problem take shape.

$$\frac{\partial \mathcal{L}}{\partial x}(x^{*}, \lambda) = \frac{\partial}{\partial x}\left( \frac{1}{2}x^T\mathcal{I}x + \lambda^T(Cx-d)\right) = \mathcal{I}x^* + C^T\lambda = \vec{0}$$
$$\frac{\partial \mathcal{L}}{\partial \lambda}(x^{*}, \lambda) = \frac{\partial}{\partial \lambda}\left( \frac{1}{2} x^T\mathcal{I}x + \lambda^T(Cx-d)\right) = Cx^* = d$$

So, solving this optimization problem is the same thing as solving the following linear system!

$$
\begin{align*} 
& \mathcal{I}x + C^T\lambda &=  \vec{0} \\\\ 
& Cx  &=  d
\end{align*}
$$

The optimization problem can be solved by solving the original system or some manipulations can be made that rapidly accelerate this process.

Here is the source code for the naive version in Python 3.7 and NumPy.

```python
import numpy

def min_norm_solve_naive(C:numpy.ndarray, d:numpy.ndarray, return_multipliers:bool = True) -> numpy.ndarray:
    # get the shapes of the matrices
    num_x = C.shape[1]
    num_l = C.shape[0]
    
    # Build the system to solve
    A = numpy.block([[numpy.eye(num_x), C.T],[C, numpy.zeros((num_l,num_l))]])
    b = numpy.block([[numpy.zeros((num_x,1))],[d]])
    
    # solve linear system with numpy
    solution = numpy.linalg.solve(A, b)
      
    if return_multipliers:
        return solution
    
    return solution[:num_x]
  
```

We can rearrange the system.

$$\mathcal{I}x + C^T\lambda =  \vec{0} \rightarrow x = -C^T\lambda$$

$$Cx = d \rightarrow -CC^T \lambda = d$$

$$x = -C^T\lambda \rightarrow x= C^T(CC^T)^{-1}d$$

We can solve the multipliers system then substitute it back into the expression for x.

Here is the source code for the informed version in Python 3.7 and NumPy.

```python
import numpy

def min_norm_informed(C:numpy.ndarray, d:numpy.ndarray, return_multipliers:bool = True) -> numpy.ndarray:
    
    # solve the langrange multiplier system
    lagrange_multipliers = numpy.linalg.solve(-C@C.T, d)
    
    # substatute back to compute x
    x = -C.T@lagrange_multipliers
    
    if return_multipliers:
        return numpy.block([[x],[lagrange_multipliers]])

    return x

```

This is much faster than the naive version as it solves a much smaller system of equations. For example, with 100 constraints and 1000 dimensions, the informed version ran 80x faster than the naive version (.24 ms vs. 19.3 ms).

