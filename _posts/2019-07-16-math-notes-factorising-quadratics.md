---
title: "Math Notes: Factorising Quadratics"
date: 2019-07-16
modified: 2019-07-16
tags:
category: mathematics
published: true
---

This is one of those fundamental bits of math that I could not get my head round in school. There are three ways to do this. For the moment this only covers the simplest of the techniques.

Quadratic equations take the form $ax^2 + bx + c = 0$. Sometimes you might need to do some work to get them to this form, for example $x^2 + 4x = 6$ can be rearranged by taking 6 away from each side to get it into the correct form. Either or both of $b$ and $c$ can be 0, however $a$ cannot.

To solve the equation $f(x) = x^2 + 6x + 8$ we must first set $f(x)$ to 0 giving us the following:

$$
x^2 + 6x + 8 = 0
$$

In order to solve this we need to reverse the process of multiplying out the brackets and rendering the equation in the following form:

$$
(x + ?)(x+?) = 0
$$

To answer the ?'s in this we need to find the two numbers that sum to the co-efficient $b$ and who's product is $c$.

### Lets do this:

1.  Find the factors of 8:

	8 &divide; 1 = 8 <br>
	8 &divide; 2 = 4 <br>
	<del>8 &divide; 3 = 2.667</del> <br>
	8 &divide; 4 = 2 <br>
	<del>8 &divide; 5 = 1.6</del> <br>
	<del>8 &divide; 6 = 1.333</del> <br>
	<del>8 &divide; 7 = 1.143</del> <br>
	8 &divide; 8 = 0

	Which leaves us with 1, 2, 4, 8

1. Identify the two numbers which add up to $b$ AND have a product $c$: 2, 4
2. Solve for $x$, remembering that as $(x + 2)(x + 4) = 0$ either $x+2$ or $x+4$ or both must also equal zero:

<div style="display: flex; justify-content: space-evenly">
$$
\begin{align*}
    x + 2 &= 0 \\
        x  &= -2
\end{align*}
$$

$$
\begin{align*}
    x + 4 &= 0 \\
        x  &= -4
\end{align*}
$$
</div>

So our two answers to this equation are $x = \|2\|$ and $x = \|4\|$ where the $\|$ symbols signify absolute value irrespective of sign.

## Completing the square

Any quadratic equation $Ax^2 + Bx + C$ can be re-written in the form $A(x-h)^2 + k$ for some constants h and k.

$$
Ax^2 + Bx + C = A(x-h)^2 + k
$$

### Longform workthrough:

$$
\begin{align*}
x^2 + 6x + 13 &= A(x-h)^2 + k \\
              &= A(x^2 - 2hx + h^2) + k \\
              &= Ax^2 - 2Ahx + Ah^2 + k
\end{align*}
$$

Looking at the quadratic terms $x^2$ and $Ax^2$ must be equal, therefore $A$ must be 1. Taking the $A$'s out leaves us with:

$$
x^2 + 6x + 13 = x^2 - 2hx + h^2 + k
$$

Looking at the linear terms we can see we need to figure out what value of $h$ when multiplied by 2 will give us 6.

$$
\begin{align*}
x^2 + 6x + 13 &= x^2 - (2*-3)x + (-3)^2 + k \\
              &= x^2 - 6x + 9 + k
\end{align*}
$$

Which leaves us with $k = 4$.

### Shortcuts

That's a lot of work to do each time, however the whole process is made easier by using the following shortcuts:

<div style="display: flex; justify-content: space-evenly; align-items: center">
$$
h = \frac{-B}{2A}
$$

$$
k = C - h^2
$$
</div>

So lets solve $x^2 + 4x + 1$, we already know $A$ is 1:

$$
\begin{align*}
h &= \frac{-4}{2}
  &= -2
\end{align*}
$$

So

$$
\begin{align*}
k &= 1 - (-2)^2
  &= -3
\end{align*}
$$

Which gives us:

$$
\begin{align*}
x^2 + 4x + 1 &= A(x-h)^2 + k \\
             &= (x+2)^2 -3
\end{align*}
$$
