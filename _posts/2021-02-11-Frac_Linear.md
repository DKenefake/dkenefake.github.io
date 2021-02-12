---
layout: post
mathjax: true
title: Fractional Linear Programming with Charnes-Cooper
date: 2021-02-12
category:
  - Blog
tags:
  - Optimization
  - Math
---

Trying to optimize ratios of variables is a common problem that crops up in practice. While this fractional constraint can not be directly addressed with linear programming, it can be reformulated with the Charnes-Cooper transformation, given the following fractional linear programming problem. (Side note, this is based on a point where $d^Tx = \beta$ is not feasible. Otherwise, it would be unbounded). 

$$\min_x \frac{c^Tx+\alpha}{d^Tx + \beta}$$

$$\text{s.t. } Ax\leq b$$

We can use the variable transform to create a linear program without the fractional objective.

$$t = \frac{1}{d^Tx + \beta}$$

$$y = tx = \frac{1}{d^Tx + \beta}x$$

The resulting programming problem is just a particular linear programming problem.

$$\max_{y, t} c^Ty + \alpha t$$

$$
    \begin{align*}
        
        \begin{split}
            \text{s.t. }d^Ty + \beta t &= 1\\
            Ay - bt &\leq 0\\
            -t&\leq 0
        \end{split}
    \end{align*}
$$

The solution to the fractional optimization problem is $x = \frac{y}{t}$. I ran into this program when looking at planning problems in my studies. This transformation lets you solve fractional linear programming problems with any linear programming solver.

## Resources

[Charnes-Cooper transformation](https://onlinelibrary.wiley.com/doi/abs/10.1002/nav.3800090303)
