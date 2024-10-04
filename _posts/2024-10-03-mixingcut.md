---
layout: post
mathjax: true
title: MixingCut - Rust MAXCUT SDP solver
date: 2024-10-03
category:
  - Blog
tags:
  - Rust
  - Optimization
  - Performance
---

This post is going back to building out tools for the QUBO solver we are making, but looks at it from a different POV. Here we are looking at making a SDP solver for the SDP that arises from the SDP relaxation of a QUBO problem.

$
\begin{align*}
\min_{X} &\text{tr}(QX) \\
s.t. &X_{ii} = 1\\
&X \succeq 0
\end{align*}
$

Now this problem has been beaten to death in the literature, in that, this is the traditional SDP of the MAX-CUT problem, and the problem that gives the famous approximation result of Geomans-Williamson. In that if you have a graph, whose weights are encoded into the $Q$ matrix, with a randomized rounding procedure will give on expectation a very decent cut on the graph. Now this is helpful to us in that we can convert any QUBO problem into a MAXCUT instance and vise versa. This 'should' provide tighter relaxations to the QUBO problem then when using Quadratic Programming based approaches.

This is all good and well, but this still requires solving an SDP, which is not a fun experience to scale. However, we are very lucky, this is a very famous problem, and therefor has received quite a lot of attention (read algorithmic attacks). Meaning that there are many published solutions on the properties and behaviors on the solution of these problems. One of the most famous results on the properties of these SDP problems is that of Burer and Monteiro. Where, they one look at a closely related problem, where we factorize $X = YY^T$, with $Y$ being a tall-skinny matrix. Now, thru this, any choice of $Y$ will result in an $X$ that is positive semidefinite, so it lets us remove the positive semidefinite. The main statement that Burer and Monteiro made is that, the rank of $X^*$ is typically very small and is no greater then $\sqrt(2n)$, so we should use that to reduce the size of $Y$ and thus make our problem easier to solve.

$
\begin{align*}
\min_{X} &\text{tr}(QYY^T) \\
s.t. &||Y_{i}||_2 = 1\\
\end{align*}
$

However, an obvious downside of doing this is that the problem is non convex. However newer results have shown (for almost any choice of $Q$) that choosing $Y$ to have more columns then $\sqrt{2n}$ will guarantee that any local minimum of this nonconvex problem is the global minimum. So we have a guild that says, we can apply local optimization algorithms to the factorized problem, and then with a fairly strong guarantee that we will arrive at the global solution of the original problem.

So, if you looked at the previous blog posts you might have some insight on what type of local algorithm I am going to use for this problem. If your guess, is "Projected Gradient Decent", then you are absolutely correct. This algorithm works very well in that the projection operator is cheap operation, in that you just normalize all of the rows of $Y$.

$$
P_{k+1} = Y_k - \alpha_k \nabla  f(Y_k)
Y_{k+1} =  \text{Proj}_ {\mathcal{C}}(P_{k+1})
$$

Given all of this, I don't think it isn't that interesting to have a long winded post about the properties of a specific type of SDP (but I have many friends that would disagree with me). So, I write this as an open source Rust program, named [MIXINGCUT](https://github.com/DKenefake/MixingCut). It takes advantage of sparse linear algebra and optimized routines for this class of problem. Currently, I the command line interface is complete and I will be making it available as a rust crate, so that it can be imported and used as a dependency of other software (with my QUBO solver Hercules being the first consumer).

So to answer question 1, does it give tighter relaxations then the QP relaxation? We will look at the solving the problem instance mk487a. This problem is based on a real world case study of radio wave interference in Italy.

If we look at the SDP relaxation, then it is approximately -1121225, and the rounded solution from the SDP generates a rounded solution of value -1105492. Meaning on the root note we get a gap of 1.4% if we use the SDP relaxation. However, if we use the QP relaxation we get a value of -36942642, and a rounded solution of -1072445. Which is a gap of 3344%. Needless to say, this is a MUCH worse gap then the SDP solution. So objective one is achieved, in that, by using this we can create much better relaxation and by doing the Geomans-Williamson rounding we get much better solutions then using QP relaxations. The actual optimal solution here is -1110926.

So that leaves the question, why build this? Why not use a reliable off the shelf solver for things like this, such as SCS? well the simple answer is that with the use of a specialized algorithm for a specialized problem we can solve these problems much faster. For the mk487a instance using MAXCUT with default settings it solved in 1.13 seconds when using SCS it times out at 1800 seconds. 

So to wrap things, up, a new solver for SDP relaxations of MAXCUT created, it is much faster then general SDP solvers (for this problem type), and can be integrated into the Hercules in a realtively easy manner without needing to create an FFI bridge.
