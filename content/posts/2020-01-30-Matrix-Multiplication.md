---
title: "Matrix Calculus"
date: 2020-01-30T20:17:54-05:00
draft: false
markup: mmark
---

Matrix calculus is very important in machine learning and control theories. However there are all kinds of rules making it hard to remember and use. In this post, I summarize some basic rules and an important proof in machine learning theory, which includes the famous "Trace Trick"! This trick is super useful and super hard as well! Haha, this is the reason why we always explore new things, because it is hard!

<!--more-->

## Differentiation

For the rules of matrix differentiation, all we need to remember are:

* Gradient
* the Jacobian matrix
* the Hessian matrix

\* All vectors in this post refer to column vectors with dimension $$N$$.

### Getting started

So for a warm up part, we differentiate a matrix with respect to a scalar variable, the result is pretty intuitive and natural:

$$\dot{A}=\begin{bmatrix}
\dot{A}_{11} & \cdots & \dot{A}_{1n} \\
\vdots & \ddots & \vdots \\
\dot{A}_{n1} & \cdots & \dot{A}_{nn} 
\end{bmatrix}$$

### Gradient

The gradient of a scalar function can be visualized as the changes in different directions. 

$$\nabla f(\textbf{x})=\begin{bmatrix}
\frac{\partial f}{\partial x_1} \\
\frac{\partial f}{\partial x_2} \\
\vdots \\
\frac{\partial f}{\partial x_n} \\
\end{bmatrix}$$

It is the most simplifed case, remember it returns a vector that's in the same shape as the vector variable.

### Jacobian Matrix

Jacobian matrix is the first-order derivative of a **vector function**, a vector function is a function that **receives a vector value and returns a vector value**.

Jacobian matrix is obtained from $$(\nabla \textbf{f})^T$$, they are equivalent.

$$\frac{\partial \textbf{y}}{\partial \textbf{x}}=[\frac{\partial \textbf{f}}{\partial x_1} \cdots \frac{\partial \textbf{f}}{\partial x_n}]=\begin{bmatrix}
\frac{\partial f_1}{\partial x_1}  & \cdots & \frac{\partial f_1}{\partial x_n} \\
\vdots & \ddots & \vdots \\
\frac{\partial f_n}{\partial x_1} & \cdots & \frac{\partial f_n}{\partial x_n}
\end{bmatrix}$$

We did gradient operation at first, transposed the result, then applied the simple first derivative expansion as discussed in the warm up section.

## Hessian Matrix

Hessian matrix is the second-order partial derivatives of a **scalar-valued** function. A scalar-valued function is a function that **receives a scalar or vector and returns a scalar**. Here we assume it the function receives a vector.

Since it is a scalar function, first we find the gradient:

$$\frac{\partial y}{\partial \textbf{x}}=\begin{bmatrix} \frac{\partial y}{\partial x_1} \\
\vdots \\
\frac{\partial y}{\partial x_n} \\
\end{bmatrix}=\nabla \textbf{f}$$

And we need to do this one more time.A sharp eye reader may notice the second derivative is the Jacobian matrix of the gradient, which is: 

$$\textbf{J}(\nabla \textbf{f})$$:

However, **Hessian matrix is defined with an extra tranpose operation**, so it is actually:

$$\textbf{J}(\nabla \textbf{f})^T$$:


$$\mathbf{H}=\left[\begin{array}{cccc}
{\frac{\partial^{2} f}{\partial x_{1}^{2}}} & {\frac{\partial^{2} f}{\partial x_{1} \partial x_{2}}} & {\cdots} & {\frac{\partial^{2} f}{\partial x_{1} \partial x_{n}}} \\
{\frac{\partial^{2} f}{\partial x_{2} \partial x_{1}}} & {\frac{\partial^{2} f}{\partial x_{2}^{2}}} & {\cdots} & {\frac{\partial^{2} f}{\partial x_{2} \partial x_{n}}} \\
{\vdots} & {\vdots} & {\ddots} & {\vdots} \\
{\frac{\partial^{2} f}{\partial x_{n} \partial x_{1}}} & {\frac{\partial^{2} f}{\partial x_{n} \partial x_{2}}} & {\cdots} & {\frac{\partial^{2} f}{\partial x_{n}^{2}}}
\end{array}\right]$$

So till now I hope you can write the steps how we get Jacobian and Hessian matrix.

## Integral
And calculating the integral of the whole matrix is simple as the warm up part.

$$\int A(t) {\rm d}t=\begin{bmatrix}
\int A_{11}{\rm d}t & \cdots & \int A_{1n}(t){\rm d}t \\
\vdots & \ddots & \vdots \\
\int A_{n1}(t){\rm d}t & \cdots & \int A_{nn}(t){\rm d}t
\end{bmatrix}$$

## Trace trick

In this section, we will teach you how to understand trace trick and as usual, we start with a simple case:

### Derivative of a quadratic form

**If you have no idea what a quadratic form is, I suggest you stop reading downwards**, because you have to have a good understanding of it, otherwise you will not realize the reason and the true importance of finding its derivative.

I also suggest a wonderful book for that foundation, *Linear Algebra and its Applications*, this is a must-read book for new learners.

So here we begin!

Define a function:

$$f(\textbf{x})=\textbf{x}^T\textbf{Ax}$$

This is a scalar function, since $$\textbf{x}$$ is $$m\times 1$$. So the size of the output is:

$$(1\times m)\times(m\times m)\times(m\times 1)=(1\times 1)$$

And we need to find its first derivative w.r.t $$\textbf{x}$$. How?

You may think, it is same as finding the gradient of the function, so just expand the equation and find partial derivative.

Yes, it is indeed a good idea, but expanding all the terms is way to tedious and humans are fallible. Is there any other way around?

Yes, there is. It is called **Trace Trick** and is widely used in finding the derivative of quadratic form function.

### Trace
From wikipedia:

> In linear algebra, the trace of a square matrix $$A$$ is defined to be the sum of elements on the main diagonal of $$A$$. The trace of a matrix is the sum of its eigenvalues, and it is invariant with respect to a change of basis.

### Trace laws
There are two most important laws:

#### Transpose law
$$\operatorname{tr}(\mathbf{A})=\operatorname{tr}\left(\mathbf{A}^{\top}\right)$$

#### Cyclic permutation law
$$\operatorname{tr}(\mathbf{A B C D})=\operatorname{tr}(\mathbf{B C D A})=\operatorname{tr}(\mathbf{C D A B})=\operatorname{tr}(\mathbf{D A B C})$$

How do we apply the trace trick in our calculation? Well, from the definition of the trace, a trace of a scalar (which can be considered as a $$1\times 1$$ matrix) is itself, so:

$$\operatorname{tr}(\texbf{x}^T\textbf{Ax})=\texbf{x}^T\textbf{Ax}$$

Now it is the trick time:

$$\begin{aligned}
{\rm d}(\textbf{x}^T\textbf{Ax}) &={\rm d}(\operatorname{tr}(\textbf{x}^T\textbf{Ax})) \\
&=\operatorname{tr}({\rm d}(\textbf{Ax}\textbf{x}^T)) \\
&=\operatorname{tr}({\rm d}(\textbf{Ax})\textbf{x}^T+{\bf Ax}{\rm d}(\textbf{x}^T)) \\
&=\operatorname{tr}({\rm d}(\textbf{Ax})\textbf{x}^T)+\operatorname{tr}({\bf Ax}{\rm d}(\textbf{x}^T)) \\
&=\operatorname{tr}(\textbf{A}({\rm d}\textbf{x})\textbf{x}^T)+\operatorname{tr}\left((({\rm d}\textbf{x})\textbf{x}^T \textbf{A}^T)^T\right) \\
&=\operatorname{tr}((\textbf{x}^T\textbf{A}({\rm d}\textbf{x})) + \operatorname{tr}(\textbf{x}^T\textbf{A}^T({\rm d}\textbf{x})) \\
&=\operatorname{tr}(\textbf{x}^T\textbf{A}){\rm d}\textbf{x} + \operatorname{tr}(\textbf{x}^T\textbf{A}^T){\rm d}\textbf{x} \\
&=\operatorname{tr}\left((\textbf{x}^T\textbf{A}+\textbf{x}^T\textbf{A}^T){\rm d}\textbf{x}\right) \\
&=(\textbf{x}^T\textbf{A}+\textbf{x}^T\textbf{A}^T){\rm d}\textbf{x}
\end{aligned}$$

We used cyclic permutation law at beginning and at the end twice, and used transpose law only once to transform $${\rm d}\textbf{x}^T$$

And we can get the result $$\textbf{x}^T\textbf{A}+\textbf{x}^T\textbf{A}^T$$ simply in several steps!

## Differetiate an inverse

$$\frac{\rm d}{{\rm d}t}(A^{-1})=-A^{-1}\dot{A}A^{-1}$$



