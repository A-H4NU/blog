---
title:  Secure Multi-Party Computation (1)
date: 2025-07-27 3:06:00 +0900
categories: [Cryptography]
tags: english mpc
toc: true
math: true
---

<!--toc:start-->
- [Introduction](#introduction)
- [Sharing Secrets](#sharing-secrets)
- [Linearity of Secret Sharing](#linearity-of-secret-sharing)
  - [Addition of Two Secrets](#addition-of-two-secrets)
  - [Scalar Multiplication of Secrets](#scalar-multiplication-of-secrets)
- [Example: Calculating the Average of Test Scores](#example-calculating-the-average-of-test-scores)
- [Conclusion](#conclusion)
<!--toc:end-->

## Introduction

**Secure Multi-Party Computation** (SMC or MPC) is a subfield of cryptography that
enables multiple parties to jointly compute a function over their private inputs
_without_ revealing those inputs to one another. The only information revealed is
the final output of the computation.

A classic illustration is [Yao's Millionaires'
Problem](https://en.wikipedia.org/wiki/Yao's_Millionaires'_problem). Two
millionaires, Alice and Bob, want to determine who is richer. However, neither
wants to reveal their actual net worth. An MPC protocol allows them to learn the
result—for instance, a single bit indicating if Alice's wealth is greater than
Bob's—without disclosing the underlying numerical values. If Alice has wealth
$$W_A$$ and Bob has wealth $$W_B$$, they compute a function

$$f(W_A, W_B) = \begin{cases}
1 & \text{if } W_A > W_B \\
0 & \text{if } W_A \leq W_B
\end{cases}$$

without either party learning the other's input.

## Sharing Secrets

At the heart of many MPC protocols is **secret sharing**. To make these schemes
secure, all arithmetic is performed within a [finite
field](https://en.wikipedia.org/wiki/Finite_field), typically the set of
integers modulo a large [prime](https://en.wikipedia.org/wiki/Prime_number)
$$p$$, denoted as $$\mathbb{Z}_p$$.[^1] This ensures that share values "wrap
around" and do not leak information about the magnitude
of the secret.

> The notation $$\mathbb{Z}_p$$ represents the set of integers $$\{0, 1, \cdots,
> p-1\}$$, where all arithmetic operations "wrap around" a prime modulus $$p$$.
> For example, in $$\mathbb{Z}_{307}$$, the sum $$157+240$$ is not $$397$$, but
> rather $$96$$, which is the remainder when $$397$$ is divided by $$307$$.
> Because $$p$$ is prime, this system forms a finite field, which guarantees
> that standard operations like addition, multiplication, and even division
> (except by zero) are well-defined.
{: .prompt-info }

With **additive secret sharing**[^2], a secret value $$S$$ is split into $$n$$
"shares" among $$n$$ participants. The core properties are:
1.  Any individual share provides no information about $$S$$.
1.  All $$n$$ shares combined reveal $$S$$.

To share a secret $$S \in \mathbb{Z}_p$$ among $$n$$ parties, the party holding
$$S$$:
1.  Generates $$n-1$$ random numbers $$r_1, r_2, \dots, r_{n-1}$$ uniformly from
    $$\mathbb{Z}_p$$.
1.  Distributes these as shares to the first $$n-1$$ parties.
1.  Keeps the final share for themselves: $$s_n = (S - \sum_{i=1}^{n-1} r_i)
    \pmod p$$.

To reconstruct the secret, all $$n$$ parties reveal their shares, which are then
summed modulo $$p$$: $$\sum_{i=1}^{n} s_i = S \pmod p$$.

## Linearity of Secret Sharing

Additive sharing is powerful because it is **linear**. This property allows
participants to perform addition and scalar multiplication on their shares
locally, which corresponds to the same operation on the underlying secrets, all
*without* any communication.

### Addition of Two Secrets

For instance, if Alice and Bob each hold shares of two secrets, say $$S$$ and
$$T$$. Alice has $$s_1$$ and $$t_1$$, and Bob has $$s_2$$ and $$t_2$$ so that
$$S = s_1 + s_2$$ and $$t = t_1 + t_2$$. Now, if Alice put $$u_1 = s_1 + t_1$$
and $$u_2 = s_2 + t_2$$, then they now have a share of the sum of the two
secrets $$U = S + T$$:

$$U = S + T = (s_1 + s_2) + (t_1 + t_2) = (s_1 + t_1) + (s_2 + t_2) = u_1 + u_2$$

Note that there was _no_ communication between Alice and Bob to compute this
sum. Hence, additive sharing allows for **local computation** of sums of
secrets.

### Scalar Multiplication of Secrets

Moreover, additive sharing also allows for scalar multiplication. If Alice has
$$s_1$$ and Bob has $$s_2$$ for a secret $$S$$, they can compute a share of $$T
= \alpha S$$ where $$\alpha$$ is a public constant. To do this, Alice computes
$$t_1 = \alpha s_1$$ and Bob computes $$t_2 = \alpha s_2$$. They now have shares
of the secret $$T$$:

$$T = \alpha S = \alpha (s_1 + s_2) = \alpha s_1 + \alpha s_2 = t_1 + t_2$$

Hence, they can locally compute a share of the scalar multiplication of
secrets without any communication.

## Example: Calculating the Average of Test Scores

With enough background, we can now illustrate how additive sharing can be used
in a simple MPC protocol. Suppose Alice, Bob, and Charlie want to compute the
average of their test scores without revealing their individual scores. If there
are only two people, knowing the average is equivalent to knowing the other
person's score, so we have to have at least three people to make it interesting.

### Settings

Suppose three participants, Alice, Bob, and Charlie, have test scores in the
range of $$0$$ to $$100$$. The maximum possible sum is $$100+100+100=300$$. We
must choose a prime modulus $$p$$ larger than this maximum sum. Let's choose
$$p=307$$. Hence, our base field is $$\mathbb{Z}_{307}$$ to
represent the scores. Let $$A$$, $$B$$, and $$C$$ be the test scores of Alice,
Bob, and Charlie, respectively. For illustration, let's say $$A = 60$$, $$B =
70$$, and $$C = 80$$. (Of course, the actual scores are not known to the other
participants.) We are going to calculate the sum of the scores, which is
$$A+B+C = 210$$, because it is much simpler.

### The First Step: Sharing the Scores

As discussed, each participant will share their score with the others. For
example, Alice randomly chooses $$247$$ for Bob and $$193$$ for Charlie and then
computes her share as $$(60 - 247 - 193) \mod 307 = 234$$ so that the sum of
three shares is $$60$$.

| | Alice | Bob | Charlie | Secret (Sum) |
| :--- | :---: | :---: | :---: | :---: |
| Shares of $$A$$ | $$234$$ | $$247$$ | $$193$$ | $$60$$ |
| Shares of $$B$$ | $$171$$ | $$32$$ | $$174$$ | $$70$$ |
| Shares of $$C$$ | $$188$$ | $$140$$ | $$59$$ | $$80$$ |

### The Second Step: Adding the Shares

Now, each participant has shares of their own score and the other two
participants' scores. They can compute the sum of their shares locally.
For instance, the share of $$A+B+C$$ for Alice is computed as follows:
$$(222 + 171 + 188) \mod 307 = 286$$.

| | Alice | Bob | Charlie | Secret (Sum) |
| :--- | :---: | :---: | :---: | :---: |
| Shares of $$A+B+C$$ | $$286$$ | $$112$$ | $$119$$ | $$210$$ |

Note that the sum of the shares of $$A+B+C$$ is exactly $$210$$ as desired.
As the range of the sum of the scores is from $$0$$ to $$300$$, $$210$$ is
indeed the sum of the scores of Alice, Bob, and Charlie.

### The Last step: Opening the Shares

Finally, each participant opens their shares of $$A+B+C$$. This means that each
participant reveals their share to the others. For instance, Alice reveals
$$286$$ to Bob and Charlie. With all three shares, they can compute the final
result that the average score is $$\frac{210}{3} = 70$$ but they do not know the
individual scores of the others.

## Conclusion

We've demonstrated how **Secure Multi-Party Computation** uses simple techniques
like additive sharing to let groups compute a shared result while keeping their
individual data private. This method's **linearity** allows for local
calculations, but more complex operations, like multiplying secrets or defending
against malicious users, require more advanced protocols, which we'll explore in
a future post.

Still, MPC is a cornerstone technology. It provides a powerful framework for
collaboration on sensitive data in fields from finance to healthcare without
sacrificing privacy.

[^1]: Although our examples use the prime field $$\mathbb{Z}_p$$, it's not the only option for the base set. Depending on the needs of the computation, it can be more convenient to build the protocol over other finite fields or [rings](https://en.wikipedia.org/wiki/Finite_ring).
[^2]: Of course, there are many other secret sharing schemes, such as [Shamir's secret sharing](https://en.wikipedia.org/wiki/Shamir's_secret_sharing), but we will focus on additive sharing for now.
