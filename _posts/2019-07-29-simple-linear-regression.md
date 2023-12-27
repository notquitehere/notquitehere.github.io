---
title: "Math Notes: Simple Linear Regression"
date: 2019-07-29
modified: 2019-07-29
tags:
category: Mathematics
published: true
---

Simple linear regression is the process of identifying the linear function (non-vertical straight line) which best describes the relationship between one independent and one dependent variable. In order to do this we need a way to assess the fit, the simplest way to do this is to measure the difference between the actual values and the predicted ones, however if we just take the raw measurements we might find the positives and negatives cancel each other out so we're going to take the squares. Our aim here is to minimise the "loss" ($\mathcal{L}$) of the function.

Since we know we're looking for a non-vertical straight line we know we're looking for a function in form $mx + b$, in order to find the best fit line we need to work out what the m and b should be.

![Graph with 4 points (x1,y1 to xn,yn)](/images/lin_reg1.jpg)

For an arbitrary line the loss can be calculated by summing the differences between the actual and computed values:

$$
\mathcal{L}_{LSE} = (y_1 - (mx_1 + b))^2 + (y_2 - (mx_2 + b))^2 + (y_3 - (mx_3 + b))^2 + \ldots + (y_n - (mx_n + b))^2
$$

Muliplying out the squares gives us:

$$
\mathcal{L}_{LSE} = y_1^2 - 2y_1(mx_1 + b) + (mx_1 + b)^2 + y_2^2 - 2y_2(mx_2 + b) + (mx_2 + b)^2 + y_3^2 - 2y_3(mx_3 + b) + (mx_3 + b)^2 + \ldots + y_n^2 - 2y_n(mx_n + b) + (mx_n + b)^2
$$

Lets get rid of the rest of the brackets now:

$$
\begin{align*}
\mathcal{L}_{LSE} = &y_1^2 - 2y_1 mx_1 - 2y_1 b + m^2 x_1^2 + 2mx_1 b + b^2 + \\
                    &y_2^2 - 2y_2 mx_2 - 2y_2 b + m^2 x_2^2 + 2mx_2 b + b^2 + \\
                    &y_3^2 - 2y_3 mx_3 - 2y_3 b + m^2 x_3^2 + 2mx_3 b + b^2 + \\
                    &\ldots + \\
                    &y_n^2 - 2y_n mx_n - 2y_n b + m^2 x_n^2 + 2mx_n b + b^2
\end{align*}
$$

There are a lot of repetitive terms here. We can gather them to make this a bit cleaner:

$$
\mathcal{L}_{LSE} = (y_1^2 + y_2^2 + y_3^2 + \ldots + y_n^2) -
                    2m(y_1x_1 + y_2x_2 + y_3x_3 + \ldots + y_nx_n) -
                    2b(y_1 + y_2 + y_3 + \ldots + y_n) +
                    m^2(x_1^2 + x_2^2 + x_3^2 + \ldots + x_n^2) +
                    2mb(x_1 + x_2 + x_3 + \ldots + x_n)
$$

Now that we're here we can make this tidier by observing that

$$
\frac{(y_1^2 + y_2^2 + y_3^2 + \ldots + y_n^2)}{n} = \overline{y^2}
$$

so

$$
y_1^2 + y_2^2 + y_3^2 + \ldots + y_n^2 = n\overline{y^2}
$$

which means we can replace all of these repetitive terms with $n$ multiplied by the mean of the term:

$$
\mathcal{L}_{LSE} = n\overline{y^2} - 2m(n\overline{xy}) - 2b(n\overline{y}) + m^2(n\overline{x^2}) + 2mb(n\overline{x}) + b^2
$$

## Finding m and b

Now we can use our knowledge of the loss of the best fit line to work out what the best fit line would be by taking partial derivatives with respect to $m$ and $b$.

### m

Take partial derivative with respect to m and set to zero:

$$
-2n\overline{xy} + 2n \overline{x^2} m + 2nb\overline{x} = 0
$$

Divide through by $2n$:

$$
-\overline{xy} + \overline{x^2} m + b\overline{x} = 0
$$

add $\overline{xy}$ to each side to get the equation into the standard format

$$
\overline{xy} = m\overline{x^2} + b
$$

and finally divide by $\overline{x}$ to get rid of that $\overline{x^2}$

$$
\frac{\overline{xy}}{\overline{x}} = m\frac{\overline{x^2}}{\overline{x}} + b
$$

### b

Take partial derivative with respect to b and set to zero:

$$
-2n\overline{y} + 2nm\overline{x} + 2nb = 0
$$

Divide through by $2n$:

$$
-\overline{y} + m\overline{x} + b = 0
$$

add $\overline{y}$ to each side to get the equation into the standard format

$$
\overline{y} = m\overline{x} + b
$$

## Find m and b

Now that we have two points which we know are on the best fit line:

$$
\frac{\overline{xy}}{\overline{x}} = m\frac{\overline{x^2}}{\overline{x}} + b
$$

and

$$
\overline{y} = m\overline{x} + b
$$

We can identify the value of $m$.

## Find m

Taking the second equation away from the first leaves us with:

$$
\begin{align*}
m\overline{x} + b &= \overline{y} \\
-m\frac{\overline{x^2}}{\overline{x}} - b &= -\frac{\overline{xy}}{\overline{x}}
\end{align*}
$${:style="margin-bottom: 3em;"}

$$
m(\overline{x} - \frac{\overline{x^2}}{\overline{x}}) = \overline{y} -\frac{\overline{xy}}{\overline{x}}
$$

which, if we solve for m gives:

$$
\begin{align*}
m &= \frac{\overline{y} - \frac{\overline{xy}}{\overline{x}}}{\overline{x} - \frac{\overline{x^2}}{\overline{x}}} \cdot \frac{\overline{x}}{\overline{x}} \\
  &= \frac{\overline{x} \cdot \overline{y} - \overline{xy}}{(\overline{x})^2 - \overline{x^2}}
\end{align*}
$$

# Find b

We can now substitute this back into our first equation: $m\overline{x} + b = \overline{y}$ in order to find b.
