---
layout: post
mathjax: true
title: Sampling Polytopes with Hit-and-Run 
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

As a quick addendum to the [last post](https://dkenefake.github.io/blog/SamplingConvex), I have written up a specialized hit-and-run samlper for full dimensional polytopes $\mathcal{P} = \{x\in\mathcal{R}^n: Ax\leq b\}. For this case, we can drive ```extent``` directly from the polytope bounds.

$$Ax\leq b\rightarrow A(x+td)\leq b\rightarrow t  = \min_{i\in\{1,\dots, n\}, (Ad)_i \geq 0} \frac{b_i - (Ax)_i}{(Ad)_i} $$

With this new rule, we no longer have to do an iterative procedure to find the extend, meaning we only have two matrix-vector multiplications. In python, the ```extent``` function is the following.

```python
def find_extents(A, b, d, x):
    
    orth_vec = A@d
    point_vec = A@x
    
    dist = float('inf')
    
    for i in range(A.shape[0]):
        
        if orth_vec[i] <= 0:
            continue
        
        dist = min(dist, (b[i] - point_vec[i])/orth_vec[i])
            
    return dist
```

For polytopes, it appears that hit-and-run sampling is somewhat more fragile, so I have added a check to make sure that we stay inside the polytope.

```python
def hit_and_run(p, x_0, n_steps:int = 10):
    
    for i in range(n_steps):
        #generate a random direction
        random_direc = random_direction(x_0.size)

        #find the extend in the direction of the random direction and the opposite direction 
        extent_forward = find_extents(p.A, p.b, random_direc, x_0)
        extend_backward = find_extents(p.A, p.b, -random_direc, x_0)
        
        #sample a delta x from the line
        pert = numpy.random.uniform(-extend_backward, extent_forward)*random_direc
        
        #check if still inside polytope
        if not numpy.all(p.A@(x_0 + pert)<= p.b):
            continue
        
    
        #make step
        x_0 =  pert + x_0
   
   return x_0 
```

Now for the benchmarking! I will also compare with the Numba and Numba + ```prange``` versions. We are not longer scaling well with the standard ```prange```! This is since the subtasks are far too quick to be parallelized effectively. I tried blocking the individual hit-and-run samples as bundles to fix this issue, but I saw no significant speedup. The lack of scaling might be a python language limitation at such small timescales. For the case of hit-and-run sampling on a polytope, we went from 87 msec to 33 usec; this is a speedup of 2600x. In other worse, instead of a calculation taking a month, it now takes ~17 min. This result stresses the importance of using the correct tool for the job, and sometimes making specialized tools is worth it.

```
hit_and_run w/o Numba                       -> 822 us per sample
hit_and_run with Numba                      ->  33 us per sample
hit_and_run with Numba + prange             ->  33 us per sample
hit_and_run with Numba + prange + blocking  ->  33 us per sample
```
