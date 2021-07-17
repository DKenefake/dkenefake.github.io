---
layout: post
mathjax: true
title: Evaluate Fourier Series Faster
date: 2021-5-25
category:
  - Blog
tags:
  - Math
  - Performance
---

This is a bit of a call back to an [older blog](https://dkenefake.github.io/blog/Fourier_Series_Acceleration) post where we accelerated the Fourier series to converge at least cubically, but we are still left with evaluating an infinite Fourier series, so if we can evaluate that faster, that would be better. There is a way to eschew almost all of the trigonometry calculations when evaluating trigonometric Fourier series with one simple trick. To remember, what we are looking at here is a generic cosine Fourier series.

$$S = \sum_{n=1}^\infty a_n cos(nx)$$

The idea is based on the following identity; basically, we have a relation between the terms in the sum and the Chebyshev polynomials.
$$cos(nx) = T_n(cos(x))$$

Letting us rewrite the sum in the following way doesn't seem to fix any problem as we still have a Chebyshev series.

$$S = \sum_{n=1}^\infty a_n cos(nx) = \sum_{n=1}^\infty a_n T_n(cos(x))$$

However, this polynomial has a recursive identity known as the three-term relation (a common trait shared with all Jacobi polynomials). So we can relate the next polynomial in the series with the last with a simple multiplication and subtraction operation. We only need to evaluate $cos(x)$ to start the iteration, but we do not need to evaluate special functions after the first term.

$$y = cos(x)$$

$$T_1(y) = 1$$

$$T_2(y) = y$$

$$T_n(y) = 2yT_{n-1}(y) - T_{n-2}(y)$$

Here is an example series we will use to test on.

$$S = \sum_{n=1}^\infty \frac{cos(nx)}{n^4+1}$$

Here is an example code for evaluating the above series on an Arduino embedded board, and with benchmarking on an ESP8266 and ESP32, we can see a rough $3x$ speed improvement. On similar code in a desktop processor, we see a $20x$ speed improvement as the compiler is. This is not meant to be an example par excellence of embedded c++ code but a quick realization of the idea.

```c++
#include <Arduino.h>

double chebychev_series(double x, long upper){
  
  double t_k2 = 1.0;
  double t_k1 = std::cos(x);
  
  double b = 2.0*t_k1;
  double summand = 0.0;

  for(long i = 1; i < upper; i++){
    // generate n^4
    double n = double(i);
    
    summand += t_k1/(n*n*n*n + 1.0);
    double t_k = b*t_k1 - t_k2;

    //update the series
    t_k2 = t_k1;
    t_k1 = t_k;
  }

  return summand;
}

double cos_series(double x, long upper){

  double summand = 0.0;

  for(long i = 1; i < upper; i++){
    // generate n^4
    double n = double(i);
    summand += std::cos(n*x)/(n*n*n*n + 1.0);
  }

  return summand;
}

void setup() {
  // set up connection rate
  Serial.begin(9600);
}

void loop() {
  // put your main code here, to run repeatedly:
  double x = double(rand())/double(RAND_MAX);
  long terms = 10000L;

  int start = micros();
  double value = cos_series(x, terms);
  int end = micros();

  Serial.println(terms);
  Serial.print("Time for Cosine Series ");
  Serial.println(end - start);
  Serial.println(value);
  
  
  start = micros();
  value = chebychev_series(x, terms);
  end = micros();

  Serial.print("Time for Chebychev Series ");
  Serial.println(end - start);
  Serial.println(value);

  delay(1000);
}
```
