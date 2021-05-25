---
layout: post
mathjax: true
title: Reformulate Nonlinear Systems to Linear systems
date: 2021-5-25
category:
  - Blog
tags:
  - Regression
  - Data Driven Methods
---

It has been a while since, my last post. I have been busy! Today, I want to talk about some aspects of Koopman operator thoery (I know this sounds spookie) but it is not that bad. Basically it says that you can reformulate a nonlinear dynamical system into a (possibly) infinite dimensional linear system. However there are a lot of cases where you don't actually get an infinite dimensional reformulation of the problem!

More or less the trade off there is that you boost into a higher dimesnional space and in that higher dimensional space you can straighten out the nonlinearityies. 

Here is an example of the refomulation. This system is clearly nonlinear!

$$
\begin{pmatrix}
\dot{x_1}\\
\dot{x_2}
\end{pmatrix} = 
\begin{pmatrix}
x_1\\
\lambda - x_1^2
\end{pmatrix}
$$

However we can change this by adding in a new dimension $x_3 = x_1^2$ and substitution back in to get a linear system with only 1 extra dimension!

$$\frac{dx_3(t)}{dt} = \frac{d}{dt}x_1^2(t) = 2x_1(t)\frac{dx_1(t)}{dt}x(t)$ = 2 x_1^2(t)$

With this we can rewrite $\dot{x_2} = \lambda  - x_1^2(t) = \lambda - x_3$. And now we have a full blown linear system from out nonlinear one!

$$
\begin{pmatrix}
\dot{x_1}\\
\dot{x_2}
\end{pmatrix} = 
\begin{pmatrix}
x_1&&\\
\lambda&& - 1\\
&&2
\end{pmatrix}
\begin{pmatrix}
x_1\\
x_2\\
x_3
\end{pmatrix}
$$


