---
layout: post
mathjax: true
title: Applications of Optimization - Nonlinear Regression
date: 2021-1-3
category:
  - Blog
tags:
  - Optimization
  - Math
  - Python
  - Code
  - Regression
  - Data Driven Methods
---

I have been on a regression kick in the last few days. We have talked about multiple different classes of min norm problems, ridge regression, and least squares, now we will talk about some methods to solve nonlinear least-squares problems, aka nonlinear regression. In the previous methods, there was an assumption that the output variables were a linear function of input parameters, which isn't always the case. Population dynamics and demand response functions are usually nonlinear, and it is important to capture that information.

The Gauss-Newton Algorithm and its spirit child, the Levenberg-Marquardt Algorithm, are some of the most well-known methods to solve this problem, and they are potent tools to have in one's toolbox. The central premise of both algorithms is to repeatedly linearize the current point, approximate the local curvature of the function, find the minimum of the approximate function then move to that point. This process repeats over and over again until the parameters converge.

I will not derive the equations, but the Gauss-Newton (GN) Algorithm and the Levenberg-Marquardt (LM) Algorithm differ only in one term in the iterate. The $\lambda$ term is effectively a tradeoff term between GN and Gradient Decent (GD); as $\lambda$ increases, the steps that LM takes converge to the steps GD takes. This adds robustness to the method, at the expense of some speed (this is usually not a problem). The iterates have the following appearances, where we solve the linear systems to get the step. (Here, I am using Fletcher's refinement of LM to make the solution scale-invariant.)


$$(J^TJ)\delta_{GN} =J^T(y - \hat{y}) $$

$$(J^TJ + \lambda diag(J^TJ)) \delta_{LM} =J^T(y - \hat{y})$$

$$x^{i+1} = x^i + \delta$$

I will give an example of this in action, using GN and LM on a relatively simple model.  We have some data for the population change as a function of time, and we want to fit the following nonlinear model to it.

|    Time    |  Population  |
|:----------:|:------------:|
|      1     |      8.3     |
|      2     |     11.0     |
|      3     |     14.7     |
|      4     |     19.7     |
|      5     |     26.7     |
|      6     |     35.2     |
|      7     |     44.4     |
|      8     |     55.9     |

$$f(x, \beta_0, \beta_1) = \beta_0 e^{\beta_1 x}$$


The jacobian has the following form, 

$$\frac{\partial r_i}{\partial \beta_0} = e^{\beta_1 x}$$

$$\frac{\partial r_i}{\partial \beta_1} = \beta_0 x e^{\beta_1 x}$$


With a starting guess of $\beta_0 = 8.3, \beta_1 = .3$ we converge to the solution in only 7 iterations with $\lambda = 0$. Since we are already near the optimal parameters, the LM algorithm's added stability is not shown here, but we can see that if we increase the LM parameter, the number of iterations needed to converge rapidly increases. 


Of course the python code is included.

```python

import numpy 
import matplotlib.pyplot as plt

y_data = numpy.array([8.3,11.0,14.7,19.7,26.7,35.2,44.4,55.9])
x_data = numpy.array([1,2,3,4,5,6,7,8])

x_0 = numpy.array([[8.3],[.3]])

def function(x, beta):
    return beta[0]*numpy.exp(beta[1]*x)

def jacobian(x, beta):
    fb1 = numpy.exp(beta[1]*x)
    fb2 = beta[0]*x*numpy.exp(beta[1]*x)
    return numpy.block([[fb1],[fb2]]).T

def residual(x, y, beta):
    res = function(x, beta)-y
    return numpy.reshape(res, (res.size, 1))

def nonlinear_least_squares(x:numpy.ndarray, jf, r, lambda_:float = 0)-> numpy.ndarray:
    error = 10**10
    x_0 = x.copy()
    for i in range(100):
        
        #linearize and get residuale
        jec = jf(x_data, x_0)
        res = r(x_data, y_data, x_0)
        
        #SSE
        new_error = res.T@res
        
        #step build approx hessian + Levenberg-Marquardt Term
        approx_hess = jec.T@jec
        LM_term = lambda_*numpy.diag(numpy.diag(approx_hess)) 
        
        #solve for step
        delta = numpy.linalg.solve(approx_hess + LM_term, -jec.T@res)
        
        #take step
        x_0 = x_0 + delta
        
        #break if converged
        if numpy.abs(error - new_error) < 10**-10:
            break

        error = new_error
        
    return x_0

```
