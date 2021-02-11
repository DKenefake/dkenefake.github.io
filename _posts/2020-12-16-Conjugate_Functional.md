---
layout: post
mathjax: true
title: Calculating the Conjugate Function
date: 2020-12-16
category:
  - Blog
tags:
  - Optimization
  - Math
---

Duality theory is an important aspect to mathematical programming, that is to say solving optimization problems. It is easy to see for linear programs, as the dual of these problems is little more than taking the transpose of the matrices and vectors involved. Shown below

The original primal linear program

$$\min_{x} b^Tx$$

$$\text{s.t. } Ax \geq c^T$$

$$x\geq 0$$

Transforms into the dual linear program

$$\max_{y} c^Ty$$

$$\text{s.t. } A^Ty \leq b^T$$

$$y\geq 0$$

However, the dual of other functions is not necessarily so easy to see and calculate. We will be covering today to solve the conjugate function, otherwise known as the Frenchel dual of a differentiable convex function. The convex and differentiable assumptions make the solution process a lot friendlier, but these types of functions are not the only functions with conjugate functions. The definition of a conjugate function is as follows. Let $ \textbf{X} $ be a normed space and let $ \textbf{X}^* $ be it is dual.

$$f^* (\zeta) = \sup_{x\in X}\{\langle \zeta,x\rangle - f(x)\}$$

Now, this can look more than a little intimidating, but it is not actually that bad. If we restrict ourselves a little more for the next example, to $\mathcal{R}$, we can clarify it. If $\textbf{X} = \mathcal{R}^n$ then $\textbf{X}^* = \mathcal{R}^n$, and the inner product for this space becomes $\zeta^Tx$. For $n=1$, the inner product is just multiplication. The first part of the process is determining the dual, and the second part is calculating the domain of the conjugate function. Examples 1, 2, and 3 are taken from Boyd's Convex optimization book.

## Example 1

$$f(x) = ax+b$$

$$f^*(\zeta) = \sup_{x\in\mathcal{R}}\{\zeta x - f(x)\} = \sup_{x\in\mathcal{R}}\{\zeta x - ax+b\}$$

We can see that $f^* $ becomes unbounded for any $\zeta\neq a$, so The domain of the conjugate function $f^* $ is just $\{a\}$ and $f^* (a) = -b$.

## Example 2

$$f(x) = -\log(x)$$

$$f* (\zeta) = \sup_{x\in\mathcal{R}}\{\zeta x - f(x)\} = \sup_{x\in\mathcal{R}}\{\zeta x + \log(x)\}$$

If $\zeta > 0$ we can increase $x$ arbitrarily and have an unbounded function, so $\zeta<0$ and using simple calculus to maximize the expression we get

$$\frac{\partial}{\partial x} \left(\zeta x + \log(x) \right) = 0 \rightarrow \zeta + \frac{1}{x} = 0 \rightarrow x = \frac{-1}{\zeta}$$

Plug this back into the definition of the frenchel dual and we get $f^* (\zeta) = -\log(-\zeta) - 1$ for $\zeta < 0$.

## Example 3
This example is in $\mathcal{R}^n$. The strictly convex quadtratic function. This comes up frequently in optimization.

$$f(x) = \frac{1}{2}x^TQx$$

$$f*(\zeta) = \sup_{x\in\mathcal{R}}\{\zeta^Tx - \frac{1}{2}x^TQx\}$$

We can see that the internal function is bounded above for all $\zeta$, so the range of $f^*$ is all of $\mathcal{R}^n$.

$$\frac{\partial}{\partial x} \left(\zeta^Tx - \frac{1}{2}x^TQx\right) = 0 \rightarrow \zeta + Qx= 0 \rightarrow x = Q^{-1}\zeta$$

plugging this back into the definition, we have the following

$$f^* (\zeta) = \zeta^TQ^{-1}\zeta - \frac{1}{2}(Q^{-1}\zeta)^TQQ^{-1}\zeta = \frac{1}{2}\zeta^TQ^{-1}\zeta$$

