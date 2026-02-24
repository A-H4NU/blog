---
title:  Collecting Verification Codes (2)
date: 2025-02-22 15:00:00 +0900
categories: [Math]
tags: english math probability
toc: true
math: true
---

## Introduction

In the [previous
post](https://a-h4nu.github.io/posts/collecting-verification-codes), we
discussed the expected number of steps to collect all verification codes. In this
post, we will calculate the variance of the number of log-in tries to collect all
verification codes.

## Problem Statement

(See the [previous post](https://a-h4nu.github.io/posts/collecting-verification-codes).)
Let $$S$$ be a finite set of size $$N$$. At the beginning, a set $$F$$ is
initialized to be an empty set. For each step, a random element $$r$$ is chosen
at uniformly random from $$S$$ and put to $$F$$, i.e., $$F \leftarrow F \cup \{
  r \}$$. Let $$X$$ be the random variable that represents the number of steps to
reach $$S = F$$. Then, what is the variance of $$X$$?

## Solution

Fix $$N$$ and let $$X_i$$ be the random variable for the number of steps to
choose $$r$$ which has not been chosen when there are $$i$$ out of $$N$$ elements
are not in $$F$$. Then, we have $$X = \sum_{i=1}^N X_i$$ where each $$X_i$$ are
independent. Hence, we have $$\mathrm{Var}(X) = \sum_{i=1}^N \mathrm{Var}(X_i)$$
where $$\mathrm{Var}(\cdot)$$ represents the variance.

Now, note that $$X_i$$ is the [geometric
distribution](https:////en.wikipedia.org/wiki/Geometric_distribution) with
parameter $$p_i = i/N$$. The probability mass function of the geometric
distribution $$Y$$ with parameter $$p$$ is given by

$$\mathrm{Pr}[Y=k] = (1-p)^{k-1}p\text{.}$$

### Variance of Geometric Distribution

Well, one may calculate the variance by calculating

$$\mathrm{Var}(Y) = \mathbb{E}[X^2] - (\mathbb{E}[X])^2 = \sum_{k=1}^\infty
k^2(1-p)^{k-1}p - \left(\sum_{k=1}^\infty k(1-p)^{k-1}p\right)^2$$

but I don't want to do that. Instead, I will use the following formula:

$$\mathbb{E}[X^n] = \left.\frac{\mathrm{d}^n}{\mathrm{d}t^n} \phi(t)\right|_{t=0}$$

where $$\phi(t) = \mathbb{E}[e^{tY}] = \sum_{k=1}^\infty e^{tk}(1-p)^{k-1}p$$
is the [moment generating
function](https://en.wikipedia.org/wiki/Moment-generating_function).
Calculating the moment generating function of $$Y$$, we get

$$\phi(t) = \sum_{k=1}^\infty e^{tk}(1-p)^{k-1}p = \frac{pe^t}{1-qe^t}$$

where $$q = 1-p$$. Then, we have

$$\phi'(t) = \frac{pe^t}{(1-qe^t)^2} \text{ and } \phi''(t)=\frac{pe^t [1-(qe^t)^2]}{(1-qe^t)^4}\text{.}$$

(Yes, I calculated for you.)
Now, we finally have

$$\mathrm{Var}(Y) = \mathbb{E}[Y^2] - \mathbb{E}[Y]^2 = \phi''(0) -
\phi'(0)^2 = \frac{2-p}{p^2} - \left(\frac{1}{p}\right)^2 = \frac{1-p}{p^2}\text{.}$$

### Finally Calculating the Variance

Now, we have

$$\mathrm{Var}(X_i) = \frac{1-p_i}{p_i^2} = \frac{N^2}{i^2} - \frac{N}{i}\text{.}$$

Hence, the variance we want to calculate is

$$\mathrm{Var}(X) = N^2 \cdot \sum_{i=1}^N \frac{1}{i^2} - N \cdot \sum_{i=1}^N
\frac{1}{i}\text{.}$$

In particular, the growth rate is $$\mathrm{Var}(X) \sim \zeta(2) \cdot N^2$$
where $$\zeta$$ is the [Riemann zeta function](https://en.wikipedia.org/wiki/Riemann_zeta_function).

## Conclusion

We have, $$\mathrm{Var}(X) \approx 15831.10$$ and $$\sqrt{\mathrm{Var}(X)}
\approx 125.82$$. Hence, in a high probability, I shall try $$518.74 \pm 125.82$$
times to collect all verification codes!
