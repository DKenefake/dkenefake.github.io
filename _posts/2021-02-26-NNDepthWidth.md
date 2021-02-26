---
layout: post
mathjax: true
title: Neural Net design Trade off Depth vs Width
date: 2021-2-26
category:
  - Blog
tags:
  - Python
  - Code
  - Regression
  - Data Driven Methods
---

An interesting problem has come up in my research that is equivalent  to "How well can we represent nonlinear behavior with a ReLU Neural Net." The open literature is evident that the answer is "very well." Still, I do not see much literature regarding the size budget for neural nets, as in "I can only afford 100 neurons" or some other criteria. Questions like this are essential when the neural net is used as a surrogate model for an optimization problem or embedded into microcontrollers where network size is a genuine concern.

Today's post will mostly be about looking at the depth vs. width trade-off of these constrained systems on the simplest contrived model I can think of. We are going to be looking at modeling $f(x, y) = xy, x,y \in [-1,1]$ with a neural net with 24 neurons. Intuitively, I think that the deeper networks should perform better than the narrow networks, but we will see if that pans out.

I generated 10000 random $x$, $y$ pairs and then used the following Python function to evaluate the network's effectiveness. This code is using Tensorflow 2.4.

```python
import numpy
import tensorflow as tf

def test(num_nodes, num_layers, epochs = 100):
    
    x_train, y_train = numpy.block([x_1, x_2]), y
    
    extraction = [tf.keras.layers.Dense(num_nodes, activation='relu') for i in range(num_layers)]
    
    model = tf.keras.models.Sequential([
        *extraction,
        tf.keras.layers.Dense(1)
    ])
    
    model.compile(optimizer = 'adam', loss = 'mae', metrics = ['mae','mse'])
    
    return model.fit(x_train, y_train, epochs=epochs, verbose = 0) 
```

This was then batched over all possible network configurations with constant layer sizes. There were some interesting results that I didn't expect. For example, the [ReLU dyoff](https://arxiv.org/abs/1903.06733#:~:text=The%20dying%20ReLU%20refers%20to,known%20about%20its%20theoretical%20analysis.) was very apparent in the last two network configurations with relatively deep neural nets.  However, what was surprising was that the 'optimal' network design seemed to be nearly square that neural net depth and width should be almost balanced with each other. While this specific insight is only applicable to this contrived system, I think some form of this insight can be used in other (nontrivial systems).

| Hidden Layers | Nodes per Hidden Layer | MAE * 10^3 | MSE * 10^5 |
|---------------|:----------------------:|:----------:|:----------:|
| 24            |            1           |     5.7    |     5.7    |
| 12            |            2           |     4.6    |     3.8    |
| 8             |            3           |     4.5    |     3.8    |
| 6             |            4           |     5.1    |     4.5    |
| 4             |            6           |     9.9    |    17.9    |
| 3             |            8           |     20     |     70     |
| 2             |           12           |     249    |    1101    |
| 1             |           24           |     248    |    1101    |
