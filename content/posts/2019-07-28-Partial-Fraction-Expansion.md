---
title: "Signal Processing Math - Partial Fraction Expansion Strategies"
date: 2019-07-28
draft: false
markup: mmark
---

Partial fraction expansion (also known as partial fraction decomposition) is a very basic math operation when we want to simplify a problem involving a fraction with polynomial numerator and denominator. We have some methods to handle the fundamental fraction terms like Laplace transform of a simple exponential signal is:

<!--more-->

$$F(s)=\frac{1}{a+s}$$

And the by inverse Laplace Transform we know the signal is either 

\(e^{-at}u(t)\) or \(-e^{-at}u(-t)\)

, depending on the transform is left-sided or right-sided.

But in a more complicated situation, while we are facing the transform like

$$F(s)=\frac{s+3}{s^3+7s^2+10s}$$

We want to decompose the fraction into a sum of simple fractions:

$$F(s)=\frac{A_1}{s}+\frac{A_2}{s+2}+\frac{A_3}{s+5}$$

So we can use the basic Laplace transform pairs to get the signal. But we need a way to determine the values of 

$$A_1,\ A_2 \text{ and }A_3$$

## A Simple Partial Fraction Expansion

The simple situation is the one shown above, there is a simple and straightforward method to determining the unknown coefficients

$$A_1,\ A_2 \text{ and }A_3$$

To find \(A_1\), multiply \(F(s)\) by \(s\).

$$\begin{aligned}
sF(s)&=s\frac{s+3}{s(s+2)(s+5)} \cr 
&=A_1+s\frac{A_2}{s+2}+s\frac{A_3}{s+5}
\end{aligned}$$

and then set \(s=0\).

$$\begin{aligned}
sF(s)|_{s=0}&=A_1+0\frac{A_2}{0+2}+0\frac{A_3}{0+5}=A_1 \cr
&=\frac{s+3}{s(s+5)}|_{s=0}=\frac{3}{10}
\end{aligned}$$

Likewise, we can get \(A_2\) by setting \(s=-2\), get \(A_3\)

by setting \(s=-5\).

## Special Cases of Parial Fraction Expansion

The simple partial fraction expansion can expand fraction into a sum of simpler fractions. However, ther are many situations where the expansion is not so simple.

* ***Order of numerator polynomial is not less than that of the denominator.*** Partial fraction expansion can only be performed when the order of the denominator polynomial (the bottom term of the fraction) is greater than the order of the numerator (the top term).  If this condition is not met, we must perform an extra step before continuing with the expansion.

* ***DIstinct Real Roots*** The problem solved above is described as the case of distinct, real roots.   This means that each term only appears once in the denominator, and the root of each term in the denominator is a distinct real number.  In the example above, the roots were at \(0\), \(-2\) and \(-5\).

* ***Repeated Real Roots*** Another possibility is a case of repeated roots. For example

$$F(s)=\frac{s+3}{s(s+2)^2(s+5)}$$

* ***Complex Roots*** If the denominator cannot be reduced to a product of real roots. For example

$$F(s)=\frac{s+3}{(s+5)(s^2+4s+5)}=\frac{s+3}{(s+5)(s+2+j)(s+2-j)}$$

#### Order of numerator polynomial is not less than that of the denominator

We can use long division to eliminate the high-order terms on the numerater.

#### Distinct Real Roots

Like the example done above, In general

$$F(s)=\frac{A_1}{s+a_1}+\frac{A_2}{s+a_2}+\cdot\cdot\cdot+\frac{A_n}{s+a_n}$$

$$A_i=(s+a_i)F(s)|_{s=-a_i}$$

When calculation is done by hand, it is easily to use the "cover-up" method.

#### Repeated Real Roots

For example, consider the fraction:

$$F(s)=\frac{s+3}{s(s+2)^2(s+5)}$$

The expansion for this case is:

$$F(s)=\frac{A_1}{s}+\frac{A_2}{s+2}+\frac{A_3}{(s+2)^2}+\frac{A_4}{s+5}$$

And we can proof the simple method cannot be used since \((s+2)\) appear twice, and we can only get one value by setting \(s=-2\).

There are two ways, the most favored by the texbook method uses the relationship

$$A_2=\left[\frac{d}{ds}(s+2)^2F(s)\right]_{s=-2}$$

so

$$A_2=\left.\frac{s(s+5)-(s+3)(2s+5)}{(s(s+5))^2}\right|_{s=-2}$$

While this method is elegant (and can be extended to multiply repeated roots), the differentiation of the ratio of polynomials is prone to errors.

##### Example: Cross Multiplication

$$F(s)=\frac{s+3}{s(s+2)^2(s+5)}=\frac{A_1}{s}+\frac{A_2}{s+2}+\frac{A_3}{(s+2)^2}+\frac{A_4}{s+5}$$

$$s+3=(s+2)^2(s+5)A_1+s(s+2)(s+5)A_2+s(s+5)A_3+s(s+2^2)A_4$$

$$s+3=(A_1+A_2+A_4)s^3+(9A_1+7A_2+A_3+4A_4)s^2+(24A_1+10A_2+5A_3+4A_4)s+20A_1 $$

This yields a four-by-four system of equations that can be solved for A1 through A4.


$$\begin{aligned}
A_1+A_2+A_4&=0 \cr
9A_1+7A_2+A_3+4A_4 &=0 \cr
24A_1+10A_2+5A_3+4A_4&=1 \cr
20A_1&=3
\end{aligned}$$

Or, in the matrix form

$$\begin{bmatrix}
{1} & {1} & {0} & {1} \cr
{9} & {7} & {1} & {4} \cr
{24} & {10} & {5} & {4} \cr
{20} & {0} & {0} & {0}
\end{bmatrix}\begin{bmatrix}
{A_{1}} \cr
{A_{2}} \cr
{A_{3}} \cr
{A_{4}}
\end{bmatrix}=\begin{bmatrix}
{0} \cr
{0} \cr
{1} \cr
{3}
\end{bmatrix}$$

We can combine the cover-up method to get the coefficients that is distinct then use cross multiplication to get the coefficients that corresponds to the repeated roots.

### Complex Roots

Consider the fraction

$$\begin{aligned}
F(s) &=\frac{s+3}{(s+5)\left(s^{2}+4 s+5\right)}=\frac{s+3}{(s+5)(s+2-j)(s+2+j)} \cr
&=\frac{A_{1}}{(s+5)}+\frac{A_{2}}{(s+2-j)}+\frac{A_{3}}{(s+2+j)}
\end{aligned}$$

where

$$\begin{array}{l}
{A_{1}=(s+5) F(s)|_{s=-5}} \cr
{A_{2}=(s+2-j) F(s)|_{s=-2+j}} \cr
{A_{3}=(s+2+j) F(s)|_{s=-2+j}=A_{2}^{*}}
\end{array}$$

Another way to expand the fraction without resorting to complex numbers is to perform the expansion as follows.

$$F(s)=\frac{s+3}{(s+5)(s^{2}+4 s+5)}=\frac{A}{s+5}+\frac{B s+C}{s^{2}+4 s+5}$$

Note that the numerator of the second term is no longer a constant, but is instead a first order polynomial. Use cross-multiplication to get \(A\ B\ C\).
