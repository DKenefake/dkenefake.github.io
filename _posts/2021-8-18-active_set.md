---
layout: post
mathjax: true
title: Active Set Optimization Methods
date: 2021-8-18
category:
  - Blog
tags:
  - Math
  - Optimization
  - Python
---

This will be a quick post. The active set method is based on finding the optimal 'active set' of an optimization problem. Where this active set just describes the active constraints at the optimal point. This is a fairly simple algorithm in that we start with some point and then activate and deactivate constraints until we converge to the optimal 'active set'.

Contrary to what the [wikipedia article](https://en.wikipedia.org/wiki/Active-set_method) states, we do not need a feasible starting point to initiate the algorithm if we modify it slightly. Note, this will not handle infeasible problems, and additional checks will need to be used.

$$
\begin{enumerate}
  \item Find unconstrained minimizer, assign that to x (or any vector)
  \item Add most violated constraint to active set if a violated constraint exists
  \item solve resulting KKT system
  \item remove constraints with negative Lagrange multipliers
  \item if the active set was changed in this iteration go back to 2 else optimal
\end{enumerate}
$$

This is rather straightforward to code up for specific problem types. Here we have some example code for a convex quadratic problem with inequality constraints. The code for this will be at the bottom of this post. There are much more efficient implementations of active set methods that take advantage of problem structure and matrix factorizations. This is to show the idea. 

$$
\begin{align}
\min f(x) = \frac{1}{2}x^TQx + cx\\
\text{s.t. }& Ax \leq b
\end{align}
$$


We can define and then solve a convex QP by specifying the program and calling the solve function. Here is an example of a random QP with 2000 variables and box constraints.

```python
num_vars = 2000
Q = numpy.random.randn(num_vars,num_vars)
c = numpy.random.randn(num_vars,1)
A = numpy.block([[numpy.eye(num_vars)],[-numpy.eye(num_vars)]])
b = numpy.ones((2*num_vars,1))*.05
qp = QP(Q@Q.T, c, A, b)
x, l, active_set = qp.solve_optimization()
```

#### Source for QP class

```python

import numpy
from dataclasses import dataclass

@dataclass
class QP:
    """
    Simple implementation of an active set method for a convex QP. Does not handle infeasible problems
    """
    Q: numpy.ndarray
    c: numpy.ndarray
    A: numpy.ndarray
    b: numpy.ndarray
        
    def num_vars(self) -> int:
        return self.Q.shape[1]
    
    def num_constraints(self) -> int:
        return self.A.shape[0]
    
    def unconstrained_min(self) -> numpy.ndarray:
        return numpy.linalg.solve(self.Q, -self.c)
    
    def compute_slacks(self, x) -> numpy.ndarray:
        return self.b - self.A@x
    
    def solve_kkt(self, active_set):
        """
        Solves the KKT system for a given active set
        """
        active_set = list(active_set)
        
        
        A_mat = self.A[active_set]
        b_mat = self.b[active_set]
        
        num_active_set = len(active_set)
        #form kkt matrix
        kkt = numpy.block([[self.Q, A_mat.T],[A_mat, numpy.zeros(( num_active_set, num_active_set))]]) 
        rhs_kkt = numpy.block([[-self.c],[b_mat]])
        
        sol = numpy.linalg.solve(kkt, rhs_kkt)
        
        x = sol[:self.num_vars()]
        l = sol[self.num_vars():]
        
        return x, l
    
    
    def violated_constraints(self, x):
        """
        Helper function to select violated constraints
        """
        # numerical fudge
        epsilon = 10**(-10)
        
        slacks = self.compute_slacks(x)
        
        violated_slacks = numpy.where(slacks < -epsilon)[0]
    
        if len(violated_slacks) == 0:
            most_violated = None
        else:
            most_violated = numpy.argmin(slacks)
        
        return violated_slacks, most_violated
    
    def solve_optimization(self):
        """
        Simple implmentation of active set method
        """
        x = self.unconstrained_min()
        l = numpy.array([])
        active_set = []
        
        while True:    
            # create a copy of the old active set
            old_active_set = active_set.copy()
            
            # calculate the slacks involved
            violated, most_violated = self.violated_constraints(x)

            # add most violated constraint to active set
            if most_violated is not None:
                if most_violated not in active_set:
                    active_set.append(most_violated)
            

            # solve resulting kkt system
            x, l = self.solve_kkt(active_set)
            
            # probably a better way to write this
            active_set = [active_set[index] for index, l_i in enumerate(l) if l_i > 0]
            
            # if the active set has not changed then we are converged to an optimal point
            if old_active_set == active_set:
                break
            
            active_set.sort()
            
        return x, l, active_set

```

