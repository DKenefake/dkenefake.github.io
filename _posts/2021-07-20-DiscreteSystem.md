---
layout: post
mathjax: true
title: Discretizing Linear Systems and Matrix Exponentials
date: 2021-7-20
category:
  - Blog
tags:
  - Math
  - Modeling
  - Python
  - PSE
---

This post is mostly about trying to convert continuous-time systems to discrete-time systems. This is an important step in optimal control and moving horizon estimation. This transform is to allow the use of much more standard optimization algorithms to use it. Thus, instead of an optimal control problem needing to solve a calculus of variations problem, a more standard NLP can be solved (usually this is a quadratic program (we can solve these problems with millions of variables fairly fast)).

Here is the continuous linear time-invariant state-space model; this is, in effect, a linear multivariate ODE with exogenous variables (the u terms).

$$\dot{x}(t) = Ax(t) + Bu(t)$$

$$y(t) = Cx(t) + Du(t)$$

Here is the discrete linear time-invariant state-space model; we can see that this is more or less the same structure, but the difference here is the discrete-time steps.

$$x_+ = A_dx + B_du$$

$$y = Cx + Du$$

Doing some handwaving, we need can do the following operations to generate the matrix parameters $A_d$, $B_d$. Unfortunately, these math operations are nontrivial.

$$A_d = e^{AT}$$

$$B_d = \left(\int_0^{T} e^{A\tau}d\tau\right)B = A^{-1}(A_d - \mathbf{I})B$$

However, when you try to calculate these expressions, the problem with evaluating the integral of the matrix exponential as there is very little software support for evaluating this integral if $A$ is singular. Still, many interesting systems are singular,  like the nobel double integrator. So we can build series expressions for these. There is a matrix exponential in the scipy package.

$$
\begin{align}
e^{At} &= \mathbf{I} + \sum_{n = 1}^\infty\frac{A^nt^n}{n!}\\
\int_0^T{e^{A\tau}}d\tau &= \int_0^T{\mathbf{I}}d\tau + \sum_{n = 1}^\infty\int_0^T{\frac{A^nt^n}{n!}}d\tau\\
&= T\mathbf{I} + \sum_{n = 1}^\infty\frac{A^nT^{n+1}}{(n+1)!}\\
\end{align}
$$

The code listing for these is written in Python at the bottom of the post. Care was taken so that it remains numerically stable, and was validated a for the nonsingular case with the following identity.

$$\left(\int_0^{T} e^{A\tau}d\tau\right)A + \mathbf{I} = e^{AT}$$

### Example 
Given a double integrator system defined below, we want to generate the discrete state-space system with a time step equal to 1.

$$A = \begin{pmatrix}
0 & 1\\
0 & 0
\end{pmatrix} \quad B = \begin{pmatrix}
0\\
1
\end{pmatrix}$$

To calculate the discrete $A$ matrix, $A_d$, we need to calculate the matrix exponential.

$$A_d = e^{A} = \begin{pmatrix}
1 & 1\\
0 & 1
\end{pmatrix}$$

Since our $A$ matrix is singular, we have to calculate the integral expression.

$$\int_0^1e^{A\tau}d\tau = \begin{pmatrix}
1 & 0.5\\
0 & 1
\end{pmatrix}$$

$$B_d = \int_0^1e^{A\tau}d\tau = \begin{pmatrix}
1 & 0.5\\
0 & 1
\end{pmatrix}\begin{pmatrix}
0\\
1
\end{pmatrix}= \begin{pmatrix}
0.5\\
1
\end{pmatrix}$$

### Code

```python

def matrix_exp(A:numpy.ndarray):
    # Matrix exponential e^A
    U, s, V = numpy.linalg.svd(A)
    
    lambda_max = numpy.max(numpy.abs(s))
    
    folds = int(numpy.math.log2(lambda_max))
  
    if folds > 0:
        exp = matrix_exp_(A*2.0**(-folds))
        for i in range(0, folds):
            exp = exp@exp
        return exp
    else:
        return matrix_exp_(A)
    
# helper function for the actual function evaluation
def matrix_exp_(A:numpy.ndarray):
    # slow general algorithm O(n^4) works for nondiagonalizable matrices
    summand = numpy.eye(A.shape[0])
    
    running_matrix = numpy.eye(A.shape[0])
    
    for i in range(1, 100):
        running_matrix = running_matrix @ A / i
        
        summand += running_matrix
        
        if numpy.max(numpy.abs(running_matrix)) < 10**-16:
            break
    
    return summand
    
def matrix_exp_integral(A:numpy.ndarray, T:float, cut_off:int = 200):
    # calculate integral of matrix exponential 
    # slow general algorithm O(n^4) works for nondiagonalizable matrices
    #set up the terms
    running_power = 0.5 * A * T**2
    summand = numpy.eye(A.shape[0])*T
    
    for n in range(1, cut_off):    
        # add term
        summand += running_power
        # update term
        running_power = running_power @ A * T/(n+2)
        # break condition
        if numpy.max(running_power) < (10**-16):
            break
    return summand
```






