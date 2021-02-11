---
layout: post
mathjax: true
title: Numpy Block is slow 
date: 2020-11-30
category:
  - Blog
tags:
  - Performance
  - Python
  - Code
---
I have been writing a solver that makes heavy use of Numpy. Due to error checking and other overhead in NumPy, we can avoid these costs. In a situation where one needs to compose hundreds of millions of matrices can cause performance issues.

I am comparing 3 different classes of matrix blocking.

* Blocking many single-row matrices into one matrix
* Large matrix composition


As you can see that the specialized version of NumPy matrix blocking is approximately 2 to 3 times faster!

|                |   Custom  |   Numpy  |
|----------------|:---------:|:--------:|
|  Row Matrices  |  .7 mSec  | 1.6 mSec |
| Block Matrices | 67.8 uSec | 198 uSec |



This can be run using the following script. Try it out on your computer!

```python
import numpy
row = numpy.array([[i] for i in range(10)])

row_blocking = [row for i in range(1000)]

%timeit dustin_block(row_blocking)
%timeit numpy.block(row_blocking)


brick = numpy.eye(5)

row = [brick for i in range(10)]
row_blocking = [row for i in range(10)]

%timeit dustin_block(row_blocking)
%timeit numpy.block(row_blocking)

```


```python
def dustin_block(mat_list:List[List[numpy.ndarray]]):
    if type(mat_list[0]) is not list:
        mat_list = [mat_list]

    x_size = 0
    y_size = 0

    for i in mat_list[0]:
        x_size += i.shape[1]

    for j in range(len(mat_list)):
        y_size += mat_list[j][0].shape[0]

    output_data = numpy.zeros((y_size, x_size))

    x_cursor = 0
    y_cursor = 0

    for mat_row in mat_list:
        y_offset = 0

        for matrix_ in mat_row:
            shape_ = matrix_.shape
            output_data[y_cursor: y_cursor + shape_[0], x_cursor: x_cursor + shape_[1]] = matrix_
            x_cursor += shape_[1]
            y_offset = shape_[0]

        y_cursor += y_offset
        x_cursor = 0

    return output_data
```
