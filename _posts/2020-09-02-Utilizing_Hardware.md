---
layout: post
mathjax: true
title: Utilizing Hardware - Python Numba and Cuda
date: 2020-09-02
category:
  - Blog
tags:
  - Python
  - Code
  - PRNG
  - GPU
  - Performance
---

I recently did some Monte-Carlo, and I wanted to show the differences in performance between native python code, fully parallel compiled Numpy code, and unoptimized GPU Cuda code running the cupy package. Part of what makes python so powerful is the ability to tap into these resources without changing too many things (as you will see in the code example).

I am going to use the standard estimation of pi as a benchmark of performance. This problem is defined by the following.

$$\pi = \lim_{n\to\infty}{ 4 \frac{count(x^2 +y^2 < 1)}{N}}$$

I will show the code at the end, but I just want to show the results here. (8600K @ 4.5Ghz, Nvidia 1080)  

| Method | Number of Samples | Time (s) | Million samples / sec |
|--------|:-----------------:|:--------:|:---------------------:|
| Python |    100 Million    |    43    |          2.32         |
| Numba  |    100 Million    |   .295   |          339          |
| Cuda   |    100 Million    |   .068   |          1470         |
| Numba  |     10 Billion    |   20.5   |          488          |
| Cuda   |     10 Billion    |     7    |          1428         |

By leveraging the thousands of streaming processors on the GPU, we can get an incredible speed up in our task.

I will show another example of the difference, solving a linear system of 20,000 variables in NumPy(CPU) and cupy(GPU). We will see a massive difference in performance between fp32 (float) and fp64 (double). This is due to the physical hardware having more fp32 alu cores.

| Method     |  Size | Time (s) |
|------------|:-----:|:--------:|
| Numpy      | 10000 |   2.95   |
| Cuda(fp32) | 10000 |    .49   |
| Cuda(fp64) | 10000 |   5.7!   |

We can see major performance regression when using fp64 on my consumer GPU as expected. Try out the code on your machine and see the difference! 

# Example Code - Monte-Carlo Pi Estimation

```python
from random import random
import numba
import numpy
import cupy

def monti_carlo_python(n:int, m:int)-> float:
    
    accum = 0
    
    for i in range(m):
        partial = 0
        for j in range(n):
            x = random()
            y = random()
            
            partial += x**2 + y**2 < 1
        accum += partial / n
    return 4.0*accum / m

@numba.njit(parallel=True)
def monte_carlo_cpu(n:int, m:int)-> float:
    accum = 0
    
    for i in numba.prange(m):
        x = numpy.random.random(n)
        y = numpy.random.random(n)
        
        r = x**2 + y**2 < 1.0
        
        accum += numpy.sum(r)/n
    
    return 4.0*accum/m

def monte_carlo_gpu(n:int, m:int)-> float:
    
    accum = 0
    for i in range(m):
    
        x = cupy.random.random(n, dtype=numpy.float32)
        y = cupy.random.random(n, dtype=numpy.float32)
    
        r = cupy.less(x**2 + y**2, 1.0)
        
        accum += cupy.sum(r)/n
    
    return 4.0*accum/m

%timeit monti_carlo_python(1000000, 100)
%timeit monti_carlo_numba(1000000, 100)
%timeit monte_carlo_gpu(1000000, 100)
%timeit monti_carlo_numba(1000000, 10000)
%timeit monte_carlo_gpu(1000000, 10000)
```


# Example Code - Linear System Solve

```python 
A = numpy.random.random((10000,10000))
b = numpy.random.random((10000,1))

A_gpufp64 = cupy.array(A)
b_gpufp64 = cupy.array(b)

A_gpufp32 = cupy.array(A.astype(numpy.float32))
b_gpufp32 = cupy.array(b.astype(numpy.float32))


%timeit numpy.linalg.solve(A, b)
%timeit cp.linalg.solve(A_gpufp32, b_gpufp32)
%timeit cp.linalg.solve(A_gpufp64, b_gpufp64)
```

## Note: Remember to clear out unused variables

```python
mempool = cp.get_default_memory_pool()
mempool.free_all_blocks()
```








