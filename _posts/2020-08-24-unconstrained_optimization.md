---
layout: post
mathjax: true
title: Unconstrained Optimization - Explanation and Examples
date: 2020-08-15
category:
  - Blog
tags:
  - Python
  - Code
  - Optimization
  - Math
---

# Unconstrained Optimization - Explanation and Examples 
In plain terms, optimization is the task of solving for the best option given a goal and some constraints. In today's post, we are going to look into solving convex optimization problems without constraints.  In more mathematic terms, we would read the next line as "minimize f subject to x, such that x is in $\mathcal{X}$". Where $\mathcal{X}= \mathcal{R} \text{(real numbers)}, \mathcal{R}^{n}\text{(vectors)}$ even letting $x$ be a function itself.

$$\min_{x} f(x),  \text{s.t.: } x\in \mathcal{X}$$

Unconstrained optimization crops up in many fields under different names, and understanding how to solve these types of problems is incredibly powerful. Some examples include regressing a linear function onto the data set. Here we are minimizing the sum of squared error.
 
$$\min_{\alpha_0,\alpha_1 } \sum_{i=0}^{n}\left(y_i - (\alpha_1x_i+\alpha_0)\right)^2$$

Or, more generally, we could regress any arbitrary function to a dataset in this manner. Here is an example of regressing a $k^{th}$ order polynomial to a dataset.

$$\min_{\alpha_0\in\mathcal{R}^{k} } \sum_{i=0}^{n}\left(y_i - f(x_i)\right)^2, \text{where} f(x) = \sum_{j=0}^k\alpha_j x^j$$

These types of optimization problems are typically known as **min-norm** problems. 

In a more complicated example, say we had an objective function (our goal) that took in a function, such as finding the path between two points that minimizes the length you need to travel. Here $x$ belongs to $\mathcal{L}_2$, the space of all square integrable functions. Clearly, the solution is a line between the two points, but the solution process is not intuitive.

$$\min_{x} F[x(t), x'(t), t] = \int_{x_1}^{x_2}\sqrt{1+x'(t)^2}$$

## Optimality Conditions - What makes a solution
How do we know we have a minimum of $f(x)$? We need to use necessary and sufficient conditions to guild us on this path. 
* Necessary - $A\implies B$, example "If I square an integer then it is positive". While all squared integers are positive it is not enough (sufficient) to prove that my positive integer is a perfect square. 
* Sufficient - $A\impliedby B$, to continue the example "If the square root of my number is a positive integer then my number is a perfect square". This property is sufficient to prove that my number is in fact a perfect square.
* If and only if - $A\iff B$, if it is both necessary and sufficient. A implies B, and B implies A. 

Here I will list the necessary and the sufficient conditions for unconstrained optimization (of functions with continuous first and second derivatives).

Necessary - $\nabla f(x^ * ) = 0, \nabla^2f(x^ * )\succcurlyeq 0 \implies f(x^ * )\leq f(x),x\in\mathcal{X}$

Sufficient -$\nabla f(x^ * ) = 0, \nabla^2f(x^ * )\succ 0 \iff f(x^ * )< f(x),x\in\mathcal{X}$

I will do a series of worked-out examples to show the application of these conditions. 

## Worked out example of finding a minimum of a 1D Function

For the first example, let's start easy.

$$f(x) = x^2, x\in\mathcal{R}$$

If we apply the sufficiency conditions we have $\nabla f(x) = 2x=0$ and $\nabla^2 f(x) = 2 >0$. So we can solve the first equation for  $x^ * = 0$,then check $\nabla^2 f(x^* ) > 0$ so we can state that $x^ * = 0$ is the global optimum (since this function is convex, local optimality $\implies$ global optimality).

And how we would solve it in python 3.7 using SymPy.
```python
from sympy import *
#define the varable
x = symbols(f'x')
#create the function
f = x**2
#solve the equation
solution = solve(f.diff(x),x) 
```
## Worked out example of finding a minimum of a Vector Function
Now for something more challenging. Here we have a function that takes in a vector $x$ and returns the sum of the vector components' squares.

$$f(x)= \sum_{i=0}^nx_i^2, x\in\mathcal{R}^n$$

We need to apply the conditions.

$$\nabla f(x)_i = 2x_i = 0 \rightarrow x_i = 0$$
$$\nabla^2 f(x) = 2I_n \succ 0$$

So $x^* $ is the zero vector 

And how we would solve it in python 3.7 using SymPy for the case n=3.

```python
from sympy import *
#define the variables
vars = [symbols(f'x_{i}') for i in range(3)]
#create the function
f = sum([var**2 for var in vars])
#solve the equations
solution = [solve(f.diff(var)) for var in vars]
```

## Worked out example of finding a minimum of a Vector Function (Part 2)

What if we have something more complicated? Here we are assuming that $Q$ is positive definite, and the dimensions of the vectors and matrices are consistent.

$$f(x) = \frac{1}{2}x^TQx+c^Tx, x\in\mathcal{R}^n$$

Again we apply the conditions. 

$$\nabla f(x) = Qx+c^T \rightarrow x^*=-Q^{-1}c^T$$

Here we see the first increase in computational difficulty. We have to solve a system of linear equations which can be nontrivial at large $n$. And by the fact that $Q$ is positive definite, we automatically pass the second condition.

$$\nabla^2 f(x) = Q \succ 0$$

And how we would solve it in python 3.7 using NumPy.

```python
solve_problem(Q, c):
	return -numpy.linalg.solve(Q, c.T)
```

## Worked out example of Linear Regression

Here we look at regressing the slope and intercepting a linear model onto some data $y$.

$$\min_{\alpha_0,\alpha_1 } \sum_{i=0}^{n}\left(y_i - (\alpha_1x_i+\alpha_0)\right)^2, x_i\text{ and }y_i\in\mathcal{R}$$

$$f(x) =\sum_{i=0}^{n}\left(y_i - (\alpha_1x_i+\alpha_0)\right)^2$$

Remember that we aren't taking the partial derivative with respect to $x$ but with respect to $\alpha_0$ and $\alpha_1$, because we are trying to find the optimal parameters for this regression.

$$\frac{\partial f}{\partial \alpha_0} = \sum_{i=0}^{n}2\left(y_i - (\alpha_1x_i+\alpha_0)\right) = 0$$

$$\frac{\partial f}{\partial \alpha_1} = \sum_{i=0}^{n}-2x_i\left(y_i - (\alpha_1x_i+\alpha_0)\right) = 0$$

With a little rearranging, we can see that this is the same as solving a $2\times2$ linear system.

$$\sum_{i=0}^{n}\left(y_i - (\alpha_1x_i+\alpha_0)\right) =0\rightarrow \left(\sum_{i=0}^nx_n\right)\alpha_1 +(n)\alpha_0 = \sum_{i=0}^ny_i$$

$$\sum_{i=0}^{n}x_i\left(y_i - (\alpha_1x_i+\alpha_0)\right) = 0\rightarrow\left(\sum_{i=0}^nx_n^2\right)\alpha_1+\left(\sum_{i=0}^nx_n\right)\alpha_0=\sum_{i=0}^nx_iy_i$$

And the solution is near identical in form to the previous example. We can see there that $\vec{\alpha} = Q^{-1}c$, 

And how we would solve it in python 3.7 using NumPy.

```python
import numpy
def solve_problem(x: numpy.ndarray, y:numpy.ndarray)-> numpy.ndarray:
    sum_y = numpy.sum(y)
    sum_x = numpy.sum(x)
    sum_xy = numpy.dot(x,y)
    sum_x_squared = numpy.sum(x**2)
    count = x.size
    
    Q = numpy.array([[sum_x,count],[sum_x_squared, sum_x]])
    c = numpy.array([[sum_y],[sum_xy]])
    return numpy.linalg.solve(Q, c)
```

## Worked out example of Shortest Path
We will solve the shortest path between 2 points by utilizing some aspects of calculus of variations. By a the same argument of $\nabla f(x) = 0$ for a function, we can derive the celebrated **Euler-Lagrange** Equation. For a functional of this form.

$$F = \int_a^bI[x,x'(t),t]dt$$

$$\delta F=0\rightarrow \frac{d}{dt}\left(\frac{\partial I}{\partial x'}\right) -\frac{\partial I}{\partial x}=0$$

Here, to get the solution, we need to solve a differential equation instead of a system of linear equations.

$$I[x,x'(t),t] = \sqrt{1+x'(t)^2}$$

$$\frac{\partial I}{\partial x}=0$$

$$\frac{\partial I}{\partial x'}=\frac{x'(t)}{\sqrt{1+x'(t)}}$$

substituting these back in

$$ \frac{d}{dt}\frac{x'(t)}{\sqrt{1+x'(t)}} = 0 \rightarrow \frac{x'(t)}{\sqrt{1+x'(t)}} = c$$

With a small amount of rearrangement, we see that $x'(t)=\frac{c}{\sqrt{1-c^2}}$, in other words, the function we are looking for is a straight line. I will leave the python code up to the reader :)

## Resources


* [Lecture Slides on the Unconstrained Minimization](https://wiki.mcs.anl.gov/leyffer/images/b/b3/03-unCons.pdf)
* [Video on first and second order optimality conditions](https://www.youtube.com/watch?v=65-WjzPzym0)
