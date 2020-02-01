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
* The Jacobian matrix
* The Hessian matrix

\* All vectors in this post refer to column vectors with dimension $$N$$.

### Getting started

To start with the concept, we differentiate a matrix with respect to a scalar variable, the result is pretty intuitive and natural:

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

It is the most simplifed case, remember it returns a vector that's in the same shape as the input vector variable.

### Jacobian Matrix

Jacobian matrix is the first-order derivative of a **vector function**, a vector function is a function that **receives a vector value and returns a vector value**.

Jacobian matrix is obtained from $$(\nabla \textbf{f})^{\top}$$.

$$(\nabla \textbf{f})^{\top}=[\frac{\partial \textbf{f}}{\partial x_1} \cdots \frac{\partial \textbf{f}}{\partial x_n}]=\begin{bmatrix}
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

$$\textbf{J}(\nabla \textbf{f})$$

However, **Hessian matrix is defined with an extra tranpose operation**, so it is actually:

$$\textbf{J}(\nabla \textbf{f})^{\top}$$


$$\mathbf{H}=\left[\begin{array}{cccc}
{\frac{\partial^{2} f}{\partial x_{1}^{2}}} & {\frac{\partial^{2} f}{\partial x_{1} \partial x_{2}}} & {\cdots} & {\frac{\partial^{2} f}{\partial x_{1} \partial x_{n}}} \\
{\frac{\partial^{2} f}{\partial x_{2} \partial x_{1}}} & {\frac{\partial^{2} f}{\partial x_{2}^{2}}} & {\cdots} & {\frac{\partial^{2} f}{\partial x_{2} \partial x_{n}}} \\
{\vdots} & {\vdots} & {\ddots} & {\vdots} \\
{\frac{\partial^{2} f}{\partial x_{n} \partial x_{1}}} & {\frac{\partial^{2} f}{\partial x_{n} \partial x_{2}}} & {\cdots} & {\frac{\partial^{2} f}{\partial x_{n}^{2}}}
\end{array}\right]$$

Till now I hope you can understand what Jacobian matrix and Hessian matrix are, and be able to write the steps how we get Jacobian and Hessian matrix.

## Integral
Calculating the integral of the whole matrix is as simple as the warm up part.

$$\int A(t) {\rm d}t=\begin{bmatrix}
\int A_{11}{\rm d}t & \cdots & \int A_{1n}(t){\rm d}t \\
\vdots & \ddots & \vdots \\
\int A_{n1}(t){\rm d}t & \cdots & \int A_{nn}(t){\rm d}t
\end{bmatrix}$$

## Trace trick

In this section, we will teach you what trace trick is. As usual, we start with a simple case:

### Derivative of a quadratic form

**If you have no idea what a quadratic form is, I suggest you stop reading downwards**, because you have to have a good understanding of it, otherwise you cannot realize why we want to find its derivative.

I also suggest a wonderful book for that foundation, *Linear Algebra and its Applications*, this is a must-read for all new learners.

So here we begin!

Define a function:

$$f(\textbf{x})=\textbf{x}^{\top}\textbf{Ax}$$

This is a scalar function, since $$\textbf{x}$$ is $$m\times 1$$. So the size of the output is:

$$(1\times m)\times(m\times m)\times(m\times 1)=(1\times 1)$$

And we need to find its first derivative w.r.t $$\textbf{x}$$. How?

You may think, it is same as finding the gradient of the function, so just expand the equation and find the partial derivative.

Yes, it is indeed a reasonal idea, but expanding all the terms is way to tedious and humans are fallible. Is there any other way to obtain the result without messing the matrix?

Yes, there is. It is called **"Trace Trick"** and is widely used in finding the derivative of quadratic form function.

### Trace
From wikipedia:

> In linear algebra, the trace of a square matrix $$A$$ is defined to be the sum of elements on the main diagonal of $$A$$. The trace of a matrix is the sum of its eigenvalues, and it is invariant with respect to a change of basis.

### Trace laws
Here are two most important laws:

#### Transpose law
$$\operatorname{tr}(\mathbf{A})=\operatorname{tr}\left(\mathbf{A}^{\top}\right)$$

#### Cyclic permutation law
$$\operatorname{tr}(\mathbf{A B C D})=\operatorname{tr}(\mathbf{B C D A})=\operatorname{tr}(\mathbf{C D A B})=\operatorname{tr}(\mathbf{D A B C})$$

How do we apply the trace trick in our calculation? Well, from the definition of the trace, a trace of a scalar (which can be considered as a $$1\times 1$$ matrix) is itself, so:

$$\operatorname{tr}(\textbf{x}^{\top}\textbf{Ax})=\textbf{x}^{\top}\textbf{Ax}$$

Now it is the trick time, I will write everything here, please read the following equation, and try to understand how I get to the final line:

$$\begin{aligned}
{\rm d}(\textbf{x}^{\top}\textbf{Ax}) &={\rm d}(\operatorname{tr}(\textbf{x}^{\top}\textbf{Ax})) \\
&=\operatorname{tr}({\rm d}(\textbf{Ax}\textbf{x}^{\top})) \\
&=\operatorname{tr}({\rm d}(\textbf{Ax})\textbf{x}^{\top}+{\bf Ax}{\rm d}(\textbf{x}^{\top})) \\
&=\operatorname{tr}({\rm d}(\textbf{Ax})\textbf{x}^{\top})+\operatorname{tr}({\bf Ax}{\rm d}(\textbf{x}^{\top})) \\
&=\operatorname{tr}(\textbf{A}({\rm d}\textbf{x})\textbf{x}^{\top})+\operatorname{tr}\left((({\rm d}\textbf{x})\textbf{x}^{\top} \textbf{A}^{\top})^{\top}\right) \\
&=\operatorname{tr}((\textbf{x}^{\top}\textbf{A}({\rm d}\textbf{x})) + \operatorname{tr}(\textbf{x}^{\top}\textbf{A}^{\top}({\rm d}\textbf{x})) \\
&=\operatorname{tr}(\textbf{x}^{\top}\textbf{A}){\rm d}\textbf{x} + \operatorname{tr}(\textbf{x}^{\top}\textbf{A}^{\top}){\rm d}\textbf{x} \\
&=\operatorname{tr}\left((\textbf{x}^{\top}\textbf{A}+\textbf{x}^{\top}\textbf{A}^{\top}){\rm d}\textbf{x}\right) \\
&=(\textbf{x}^{\top}\textbf{A}+\textbf{x}^{\top}\textbf{A}^{\top}){\rm d}\textbf{x}
\end{aligned}$$

I used cyclic permutation law at beginning and at the end twice, and used transpose law only once to transform $${\rm d}\textbf{x}^{\top}$$. I installed trace function on first line and uninstall the trace function on the last line, $${\rm d}\textbf{x}=\lim(\textbf{x}_1-\textbf{x}_2)\in \Bbb{R}^{N}$$, it can be proved that the expression on final line is still a scalar.

And we can get the result $$\textbf{x}^{\top}\textbf{A}+\textbf{x}^{\top}\textbf{A}^{\top}$$ simply in several steps!

#### Another Important Law

$$\frac{\partial}{\partial A}\operatorname{tr}(AB)=B^{\top}$$

This law will be used once in the next section.

## Multivariate Guassian Distribution Maximum Likelihood Estimate

So in the end, I want to calculate the MLE for a multivariate guassian distribution model with all the things discussed in this post, I hope you can enjoy it as much as I do!

#### Problem:

Given $$N$$ samples $$\{X_1,X_2,X_3,...,X_i\}$$ are drawn from a p-variate multivariate guassian distribution model i.i.d:

$$p(x | \mu, \Sigma)=\frac{1}{(2 \pi)^{p / 2}|\Sigma|^{1 / 2}} \exp \left\{-\frac{1}{2}(x-\mu)^{T} \Sigma^{-1}(x-\mu)\right\}$$

Find the MLE for $$\mu\ \Sigma$$. 

#### Solution:

Define the likelihood function $$L$$,:

$$f(\mu, \Sigma)=\frac{1}{(2\pi)^{\frac{pN}{2}}|\Sigma|^{\frac{N}{2}}}\exp{\left\{\sum_{i=1}^{N}-\frac{1}{2}(X_i-\mu)^{\top}\Sigma^{-1}(X_i-\mu)\right\}}$$

In order to simplify the problem, plus due to the fact that the multivariate guassian distribution belongs to the exponential distribution family. Instead straight differentiating the likelihood function, it is better to calculate the logarithmic differentiation of the function.

$$L(\mu, \Sigma)=\ln f(\mu, \Sigma)=\sum_{i=1}^{N}-\frac{1}{2}(X_i-\mu)^{\top}\Sigma^{-1}(X_i-\mu)-\frac{pN}{2}\ln 2\pi - \frac{N}{2}\ln|\Sigma|$$

##### Find the MLE for $$\mu$$

Find $$\mu$$ first, then $$\Sigma$$, calculate the first-order partial derivative of $$L(\mu, \Sigma)$$ w.r.t $$\mu$$.

$$\frac{\partial L(\mu, \Sigma)}{\parial \mu} = \sum_{i=1}^n - \partial \frac{1}{2}(X_i-\mu)^{\top}\Sigma^{-1}(X_i-\mu) /\partial \mu $$

And if we regard $$(X_i-\mu)$$ as $$\textbf{x}$$ in the former section, the partial derivative on the right side is simply:

$$\frac{1}{2}\sum_{i=1}^{N} (X_i-\mu)^{\top}\Sigma^{-1}+ (X_i-\mu)^{\top}(\Sigma^{-1})^T$$

Since $$\Sigma$$ is the covariance matrix, it is symmetric:

$$\begin{aligned}
\Sigma^{-1}\Sigma&=(\Sigma\Sigma^{-1})^{\top} \\
\Sigma^{-1}\Sigma&=(\Sigma^{-1})^{\top}\Sigma^{\top} \\
\Sigma^{-1}&=(\Sigma^{-1})^{\top}
\end{aligned}$

The partial derivative w.r.t $$\mu$$ in the end:

$$\frac{\partial L(\mu, \Sigma)}{\partial \mu} =\sum_{i=1}^n(X_i-\mu)^{\top}\Sigma^{-1}$$

Find its critical point, i.e. the mean of all samples:

$$\hat{\mu}=\frac{1}{N}\sum_{i=1}^{N}X_i$$

##### Find $$\Sigma$$

Since we get $$\hat{\mu}$$, and apparently the $$\mu$$ estimate does not rely on $$\Sigma$$, these two parameters are independent. We can safely subsititute all $$\mu$$ with $$\hat{\mu}$$.

$$\begin{aligned}
\frac{\partial L(\hat{\mu}, \Sigma)}{\partial \Sigma^{-1}}&=\partial\left(-\frac{N}{2}\ln|\Sigma|\right)/\partial\Sigma^{-1}-\frac{1}{2}\sum_{i=1}^{N}(X_i-\hat{\mu})(X_i-\hat{\mu})^{\top} \\
&=\partial (\frac{N}{2}\ln|\Sigma^{-1}|)/\partial \Sigma^{-1} -\frac{1}{2}\sum_{i=1}^{N}(X_i-\hat{\mu})(X_i-\hat{\mu})^{\top} \\
&=\frac{N}{2\Sigma^{-1}}-\frac{1}{2}\sum_{i=1}^{N}(X_i-\hat{\mu})(X_i-\hat{\mu})^{\top} \\
&=\frac{N}{2}\Sigma-\frac{1}{2}\sum_{i=1}^{N}(X_i-\hat{\mu})(X_i-\hat{\mu})^{\top}
\end{aligned} $$

The last term on the first line was obtained by first using the cyclic permutaion then using the extra law in the former section.

$$\frac{1}{2} \sum_{n}\left(x_{n}-\mu\right)^{\top} \Sigma^{-1}\left(x_{n}-\mu\right)=\frac{1}{2} \sum_{i=1}^{N} \operatorname{tr}\left[\left(X_{i}-\hat{\mu}\right)\left(X_{i}-\hat{\mu}\right)^{\top} \Sigma^{-1}\right]$$

Then using the extra law.

$$\frac{\partial}{\partial\Sigma^{-1}}(\frac{1}{2} \sum_{i=1}^{N} \operatorname{tr}\left[\left(X_{i}-\hat{\mu}\right)\left(X_{i}-\hat{\mu}\right)^{\top} \Sigma^{-1}\right])=\frac{1}{2}\sum_{i=1}^{N}(X_i-\hat{\mu})(X_i-\hat{\mu})^{\top}$$

