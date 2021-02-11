---
layout: post
mathjax: true
title: Picard Iteration - Theory and Application
date: 2020-09-03
category:
  - Blog 
tags:
  - Python
  - Math
  - Code
---

Picard Iteration is a neet method I stumbled across today. It is an iterative method coming from fixed-point theory to solve an initial value problem symbolically. 

The theory behind it is direct and straightforward. Given an IVP of the following form

$$y'(x) = f(x,y(x)),y(x_0)=y_0$$

We can integrate both sides and manipulate to get

$$y(x)-y(x_0) = \int_{x_0}^{x}{f(x,y(x))dx} \rightarrow  y(x)= y(x_0) +\int_{x_0}^{x}{f(x,y(x))dx}$$

With this expression, we can inject an initial guess and iteratively solve

$$\phi_0(x) = y_0$$

$$\phi_{i+1}(x) = y_0 + \int_{x_0}^{x}{f(x,\phi_i(x))dx}$$

$$\lim_{i\to\infty}{\phi_i(x)} = y(x)$$

# Example - Simple First-order decay
The equation this example is the following

$$y'(x) = -y(x), y(0) = 1$$

The following recurrence relation can be made

$$\phi_0(x) = y_0$$

$$\phi_{i+1}(x) = y_0 -\int_{x_0}^{x}{\phi_i(x)}$$

at eatch iteration we slowly build a more accurate solution

$$\phi_1(x) = 1 - x$$

$$\phi_2(x) = 1 - x + \frac{x^2}{2}$$

$$\phi_3(x) = 1 - x + \frac{x^2}{2}- \frac{x^3}{6}$$

$$\phi_n(x) = \sum_{k=0}^{n}\frac{(-1)^kx^k}{k!}$$

$$\lim_{n\to\infty}{\phi_n(x)} =  \sum_{k=0}^{\infty}\frac{(-1)^{k}x^k}{k!} = e^{-x} = y(x)$$

This is the correct answer!

Of course, doing this by hand is tedious and error-prone, so we should do it with code. 

```python 
def picard_solver(y_0, x_0, rhs_expression, iteration_count:int = 5):
    
    x, phi = sympy.symbols("x phi")
    
    phi = x_0
    
    for i in range(iteration_count+1):
        phi = y_0 + sympy.integrate(rhs_expression(x,phi), (x, x_0, x))
        
    return phi

y = picard_solver(1,0,lambda x, y: -y, n = 5)

```

# Example - logistic growth

$$y'(x) = -y(x)*(1-y(x)), y(0) = .5$$

$$\phi_0(x) = y_0$$

$$\phi_{i+1}(x) = y_0 -\int_{x_0}^{x}{\phi_i(x)*(1-\phi_i(x))}$$

We can solve this by plugging our parameters into the picard_solver function

```python
y = picard_solver(0,.2, lambda x, y: -y*(1-y), n = 5)
```

And then compare against the known solution. I used Plotly to create an interactive graph, so please zoom in and find the differences between the Picard and exact solutions. (hint: look at the edges).

```python
import numpy 
import plotly.graph_objects as go
    
    
y_set = [picard_solver(.5,0, lambda x, y: y*(1-y), n = i)  for i in range(1,6)]

x_grid = numpy.linspace(-2,2,1000)

y_picard = list()

for y in y_set:
    y_picard.append(numpy.array([float(y.evalf(subs={x:x_i})) for x_i in x_grid]))

y_exact = numpy.exp(x_grid)/(1+numpy.exp(x_grid))

fig = go.Figure()

for i,y_order in enumerate(y_picard):
    fig.add_trace(go.Scatter(x = x_grid, y = y_order, name = f"Picard Order {i+1}"))

# fig.add_trace(go.Scatter(x = x_grid, y = y_picard, name = "Picard Solution"))
fig.add_trace(go.Scatter(x = x_grid, y = y_exact, name = "Exact Solution"))

fig.show()

fig.write_html("picard_vs_exact.html")

```

{% include picardvsexact.html %}
