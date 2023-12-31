---
title: 'The Cosine Rule and Dot Product'
date: 2019-06-07
modified: 2019-06-07
tags: math, machine-learning
category: mathematics
published: true
---

The cosign rule

$$
c^2 = a^2 + b^2 - 2ab \cos\theta
$$

is a generalisation of Pythagoras' theorem $a^2 + b^2 = c^2$ to apply to all triangles rather than just right angled ones. The cosine rule reduces to Pythagoras' Theorem as $\cos 90 = 0$ (this will come in handy in a bit). As well as providing the mathematical basis behind the usefulness of the dot product for establishing the extent to which two vectors are going in the same direction.

The cosine rule allows you to work out the length of the third side of **any** triangle providing you know the length of the other two sides and the angle at which they join.

So for this arbitrary triangle

![a non-right-angled triangle](/assets/2019-06-07-arbitrary-triangle.jpg)

Where:

$$
\begin{align*}
	a &= 65 \\
	b &= 60 \\
	\theta &= 38
\end{align*}
$$

The cosign rule gives us gives us:

$$
\begin{align*}
	c^2 &= 65^2 + 60^2 - 2 * 65 * 60 * \cos38 \\
		&= 1678.516 \\
	c	&=  40.97 \\
\end{align*}
$$

You can check this by drawing the triangle.

## Vectors

Going back to vectors we know that the difference between $a$ and $b$ is $a-b$ so $c^2$ is $\|a-b\|^2$.

The pipe "\|" symbols here denote length, so $\|a-b\|$ can be read as "the length of a minus b"

![the same non-right-angled triangle as above but drawn with vectors](/assets/2019-06-07-vector-triangle.jpg)


$$
\begin{align*}
c^2 &= a^2 + b^2 - 2ab \cos\theta \\
|a-b|^2 &= |a|^2 + |b|^2 - 2 * |a| * |b| * \cos\theta
\end{align*}
$$

We know that $\|a-b\|^2$ is the dot product of (a-b) with itself: $(a-b)^T.(a-b)$. If we multiply that out we get:

$$
\begin{align*}
	(a-b).(a-b) &= a.a - b.a - b.a - b.-b \\
			   &= |a|^2 + |b|^2 - 2a.b
\end{align*}
$$

That looks familiar doesn't it?

Lets substitute that into the cosine rule:

$$
|a|^2 + |b|^2 - 2a.b = |a|^2 + |b|^2 - 2 |a| |b| \cos\theta
$$

This can be simplified:

$$
\require{cancel}
\begin{align*}
	\cancel{|a|^2} + \cancel{|b|^2} - \cancel{2}a.b &= \cancel{|a|^2} + \cancel{|b|^2} - \cancel{2} |a| |b| \cos\theta \\
	- a.b &= -|a| |b| \cos\theta
\end{align*}
$$

Multiplying through by -1 to get rid of the negative leaves us with:

$$
a.b = |a| |b| \cos\theta
$$

## Dot product {#dot-product}
What we've just shown is that the dot product tells us the extent to which the vectors we're considering go in the same direction.  This is because:

$$
\begin{align*}
	\cos 90 &= 0 \\
	\cos 0 &= 1 \\
	\cos 180 &= -1
\end{align*}
$$

So positive products tell us the vectors are going in the same direction, negative products tell us they're going in opposite directions and 0 tells us that they are orthogonal (at 90 degrees to one another). It is because of this property that we can use the dot product of a vector with itself to ascertain it's magnitude.
