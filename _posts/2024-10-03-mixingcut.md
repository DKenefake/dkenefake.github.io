---
layout: post
mathjax: true
title: MixingCut - Rust MAXCUT SDP solver
date: 2024-10-03
category:
  - Blog
Tags:
  - Rust
  - Optimization
  - Performance
---

This post returns to building out tools for the QUBO solver we are making, but from a different perspective. Here, we are looking at making an SDP solver for the SDP that arises from a relaxation of the MAXCUT problem.

$$
\begin{align*}
\min_{X} \quad &\text{tr}(QX) \\
s.t. &X_{ii} = 1\\
&X \succeq 0
\end{align*}
$$

This problem has been beaten to death in the literature, in that this is the traditional SDP relaxation of the MAX-CUT problem and the problem that gives the famous approximation result of Geomans-Williamson. If you have a graph with weights encoded into the $Q$ matrix, a randomized rounding procedure will give a decent cut on the graph, as expected. Now, this is helpful to us in that we can convert any QUBO problem into a MAXCUT instance and vice versa. This 'should' provide tighter relaxations to the QUBO problem than when using Quadratic Programming based approaches.

This is all good and well, but this still requires solving an SDP, which is not a fun experience to scale. However, we are very lucky; this is a very famous problem and, therefore, has received quite a lot of attention (read algorithmic attacks). There are many published solutions on the properties and behaviors on the solution of these problems. One of the most famous results on the properties of these SDP problems is that of Burer and Monteiro. When one looks at a closely related problem, we factorize $X = YY^T$, with $Y$ being a tall-skinny matrix. Now, through this, any choice of $Y$ will result in an $X$ that is positive semidefinite, so it lets us remove the positive semidefinite. The main statement that Burer and Monteiro made is that the rank of $X^*$ is typically very small and is no greater than $\sqrt{2n}$, so we should use that to reduce the size of $Y$ and thus make our problem easier to solve.

$$
\begin{align*}
\min_{Y}\quad &\text{tr}(QYY^T) \\
s.t. &||Y_{i}||_2 = 1\\
\end{align*}
$$

However, an obvious downside of doing this is that the problem is non-convex. However, newer results have shown (for almost any choice of $Q$) that choosing $Y$ to have more columns than $\sqrt{2n}$ will guarantee that any local minimum of this nonconvex problem is the global minimum. We have a guild that says we can apply local optimization algorithms to the factorized problem, and then with a fairly strong guarantee that we will arrive at the global solution of the original problem.

If you looked at the previous blog posts, you might have some insight into what type of local algorithm I am going to use for this problem. If your guess is "Projected Gradient Decent," then you are absolutely correct. This algorithm works very well because the projection operator is a cheap operation: You just normalize all of the rows of $Y$.

$$
\begin{align}
P_{k+1} &= Y_k - \alpha_k \nabla  f(Y_k)\\
Y_{k+1} &=  \text{Proj}_ {\mathcal{C}}(P_{k+1})
\end{align}
$$

Given all of this, it isn't that interesting to have a long-winded post about the properties of a specific type of SDP (but I have many friends who would disagree with me). So, I write this as an open-source Rust program named [MIXINGCUT](https://github.com/DKenefake/MixingCut). It takes advantage of sparse linear algebra and optimized routines for this class of problem. The command line interface is complete, and I will be making it available as a rust crate so that it can be imported and used as a dependency of other software (with my QUBO solver Hercules being the first consumer).

So, to answer question 1, does it give tighter relaxations than the QP relaxation? We will look at solving the problem instance mk487a. This problem is based on a real-world case study of radio wave interference in Italy.

If we look at the SDP relaxation, it is approximately -1121225, and the rounded solution from the SDP generates a rounded solution of value -1105492. On the root note, we get a gap of 1.4% if we use the SDP relaxation. However, if we use the QP relaxation, we get a value of -36942642 and a rounded solution of -1072445. Which is a gap of 3344%. This is a MUCH worse gap than the SDP solution. So objective one is achieved in that, by using this, we can create much better relaxation, and by doing the Geomans-Williamson rounding, we get much better solutions than using QP relaxations. The actual optimal solution here is -1110926.

So that leaves the question, why build this? Why not use a reliable off-the-shelf solver for things like this, such as SCS? The simple answer is that with the use of a specialized algorithm for a specialized problem, we can solve these problems much faster. For the mk487a instance, using MAXCUT with default settings solved in 1.13 seconds; when using SCS, it times out at 1800 seconds. 

So, to wrap things up, a new solver for SDP relaxations of MAXCUT was created. It is much faster than general SDP solvers (for this problem type) and can easily be integrated into Hercules without creating an FFI bridge. And was fun making an SDP solver that can run on a PGD based method.
