---
layout: post
mathjax: true
title: Orthoginalization - Modified Gram-Schmidt 
date: 2020-12-18
category:
  - Blog
tags:
  - Python
  - Code
  - Linear Algebra
---

Orthogonalizing a basis is an important topic in computational science and engineering, from solving linear systems to creating a basis for solving partial differential equations. The basic idea behind orthogonalization is that you want to create a new set of vectors that are all 'perpendicular' to each other to express your old vectors. The first algorithm that many people learn to do this is called the Gram-Schmidt algorithm, and while it is quite effective at dealing with many systems, the algorithm's performance to deal with high-condition number systems. This degradation is in the accuracy of the output sense and not in a run-time sense. To be clear, this code is not aiming to be performant but to be simple implementation, much more performant implementations can be found in the standard numerical linear algebra libraries.

I have included the code for a simple Numpy implementation of Gram-Schmidt. Here the row vectors of A are being orthogonalized. As a note, the usual convention is that the column vectors of A are being orthogonalized.


```python
def classical_gs(A):
    num_vecs = A.shape[0]
    
    B = numpy.zeros_like(A)
    
    for j in range(0, num_vecs):
        temp = A[j]
        for k in range(0, j):
            temp = temp - B[k].T@A[j]*B[k]
        B[j] = temp / numpy.linalg.norm(temp)
    
    return B
```

However, this suffers from floating-point error accumulation with ill-conditioned matrices. While this is not a large problem in many applications, this becomes problematic when regressing polynomials or using it as a general procedure.

This has been approved upon with the modified Gram-Schmidt algorithm, where the floating-point error builds up is mitigated much better. As you can see, this is only reordering the order of operations from the previous algorithm, and if we had exact arithmetic, the results would be the same. The performance is approximately half of the previous algorithm in run time, but the result's accuracy is much improved upon.

```python
def modified_gs(A:numpy.ndarray)->numpy.ndarray:
    num_vecs = A.shape[0]
    num_dims = A.shape[1]
    
    L = numpy.zeros(num_vecs)
    for i in range(num_vecs):
        L[i] = numpy.sqrt(A[i].T@A[i])
    
    
    V = A.copy() / L
    B = V.copy()   
    for j in range(0, num_vecs):
        B[j] = V[j]/numpy.sqrt(V[j].T@V[j])
        for k in range(j, num_vecs):
            V[k] = V[k] - (B[j].T@V[k])*B[j]    
    return B
```

# Case Study

In the regression of polynomials, it helps orthogonalize a matrix called the [Hilbert matrix](https://en.wikipedia.org/wiki/Hilbert_matrix). We can see a large difference in accuracy between the original and modified Gram-Schmidt algorithms for relatively small Hilbert matrices. For this example, we are going to orthogonalize the 10th Hilbert matrix.

This is relativley simple 

```python
from scipy.linalg import hilbert

A = hilbert(10)

Q1 = classical_gm(A)
Q2 = modified_gs(A)
```

We want to analyze the resulting orthonormal vectors to make sure that they are, in-fact, orthonormal. This is a utility function that allows us to calculate the worse failure in orthogonality and norm.

```python
def characterize_basis(Q:numpy.ndarray)-> numpy.ndarray:
    H = Q.T@Q
    err_norm = numpy.max(numpy.abs(numpy.diag(H)-1))
    
    for i in range(H.shape[0]):
        H[i,i] = 0
    
    err_orth = numpy.max(H)
    
    print(f'Worse error from ||v_i|| = 1  condition is {err_norm}')
    print(f'Worse error from othoginality is {err_orth}')
    return err_norm + err_orth
```

From the error analysis, we can see that the original algorithm has more or less completely failed. We do not have unit length vectors, nor are they even particularly orthogonal to each other. However, we have much better accuracy; while not perfect, this is acceptable for many applications. We can successively orthogonalize the resulting newly obtained basis set to correct for many numerical problems.

```
For Q1
Worse error from ||v_i|| = 1  condition is 1.1538748563146215
Worse error from othoginality is 0.758955080535941

For Q2
Worse error from ||v_i|| = 1  condition is 3.7412383962598383e-05
Worse error from othoginality is 3.9196055384436324e-05
```

# Reorthoganlizing

And you can even successively re-orthogonalize, repeatedly as an iterative process if the systems' numerics makes the roundoff error hard to deal with. Here, our example will be the 1000th Hilbert matrix, a much harder problem and extremely ill-conditioned when dealing with the Hilbert matrix. Our procedure is straightforward; we orthogonalize and iteratively orthogonalize until our error condition is met or hit an iteration limit. Here we will characterize the matrix at each intermediate step.

```python

Q = modified_gs(A)

for i in range(max_iter):
  
    # here 10**-14 is a stand in for error limit
    if characterize_basis(Q) < 2*10**-15:
        break
  
    # if we haven't hit out error tolerance, reothoginalize
    Q = modified_gm(Q)
```

We rapidly converge to an orthogonal basis.

```
For step 1
Worse error from ||v_i|| = 1  condition is 1.1332907188699446
Worse error from othoginality is 0.32705069337970993

For step 2
Worse error from ||v_i|| = 1  condition is 4.3298697960381105e-15
Worse error from othoginality is 4.2722551653253237e-14

For step 3
Worse error from ||v_i|| = 1  condition is 1.4432899320127035e-15
Worse error from othoginality is 3.0878077872387166e-16

```
