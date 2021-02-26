---
layout: post
mathjax: true
title: Sampling Convex Regions with Hit-and-Run 
date: 2021-2-25
category:
  - Blog
tags:
  - Python
  - Code
  - Math
  - Geometry
  - Performance
---

In many applications, sampling some convex region is a subproblem that needs to be done repeatedly, such as Monte-Carlo integration. This post will talk about hit-and-run sampling and apologies, but this will be a code-heavy post. Hit-and-run sampling is a straightforward algorithm where we take a point in the region, pick a random direction, then pick a new point on the line defined by that direction and point inside the region. This region does not need to be defined by linear constraints, such as a circle or an ellipse. Or even some combination of linear and nonlinear constraints. This post talks about the general convex case. There are much faster ways to sample with hit-and-run if you have information on the structure of the region, e.g. polytopic, elliptical, etc. 

```python

def hit_and_run(is_inside, x_0, n_steps:int = 10):
  
  for i in range(n_steps):
    #generate a random direction
    random_direc = random_direction(x_0.size)
    
    #find the extend in the direction of the random direction and the opposite direction 
    extent_forward = extent(is_inside, direc, x_0)
    extent_backward = extent(is_inside, direc, x_0)
    
    #sample the feasible line that we have just found
    if numpy.random.rand(1) < extend_forward / (extend_forward + extend_backward):
      x_0 = numpy.random.rand(1)*extend_forward*random_direc + x_0
    else:
      x_0 = -numpy.random.rand(1)*extent_backward*random_direc + x_0
      
  return x_0
```


One of the most critical subproblem in hit-and-run sampling is finding how far we can go in a direction before we hit a boundary. This can be done with the following approach. Here we are doing an exponential search in our random direction to find a bracket for your the farthest we can go. We then do a binary search on this bracketed distance. This process works for any convex shape.

```python

def extent(is_inside, d:numpy.ndarray, x:numpy.ndarray):
  
  #exponential search initialization
  dist = 1e-15
  
  while is_inside(dist*d + x):
    dist = 2*dist
  
  # We have a bracket for where the firthest extent we can go is [0.5*dist, dist]
  # start bisecting here
  low = 0.5*dist
  mid = 0.5*(high + low)
  high = dist
  
  while high - low > 1e-15:
    
    mid = 0.5*(high + low)
    
    if is_inside(mid*d + x):
      low = mid
    else:
      high = mid
  
  return mid
```

While this works, the performance is relatively slow. We can compile python with the Numba package to get a performance boost. We can also use Numba to parallelize this for use with the use of ```prange```. My laptop has 4 cores so that we can see excellent scaling performance. Overall using Numba made this sampling procedure ~175 times faster. This can make the difference between something taking 1 month or 4 hours. 

```
hit_and_run w/o Numba           -> 87.7 ms per sample
hit_and_run with Numba          -> 2.0 ms per sample
hit_and_run with Numba + prange -> 0.5 ms per sample
```


## Code Appendix

```python

def random_direction(n:int):
  vec = numpy.random.rand(n).reshape(n,-1)
  return vec / numpy.linalg.norm(vec, 2)

@numba.njit
def hit_and_run_parallel(is_inside, x_0, num_samples, n_steps:int = 10):
  
  samples = numpy.zeros((num_samples, x_0.size))
  
  for i in prange(num_samples):
    samples[i] = hit_and_run(is_inside, x_0, n_steps).T
  
  return samples
```
