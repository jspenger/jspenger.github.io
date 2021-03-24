---
layout: post
title: "MPC Study Group: Oblivious Transfer Extension Part 3"
date: 2021-03-17
tags: Oblivious Transfer Extension Secure Multi-Party Computation Malicious Zoom
lastupdatedon: 2021-03-17
---

<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {
  skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
  inlineMath: [['$','$']]
}
});
</script>
<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

# MPC Study Group: Oblivious Transfer Extension Part 3
This week's zoom meetup was postponed, these are some personal study notes for malicious oblivious transfer extension [1] which we partly covered [last week](oblivious-transfer-extension-part-2).

## Malicious Oblivious Transfer Extension
Last week we looked at how the semi-honest oblivious transfer protocol from [1, protocol 1] was not secure against a malicious receiver $P_r$.
In particular, the protocol can be made malicious-secure if it is ensured that the receiver always uses the same $r$ in step 2(a) in protocol 1, for the computation of $u^i = t^i \oplus G(k^1_i) \oplus r$.
We refer to this as checking the consistency of $r$, which is the topic of this post.

## Consistency Check of $r$
First, let us deal with a somewhat easier problem.
Instead of checking that *all* $u^i$ (or a sufficient amount of $u^i$) were created with the same $r$, we instead check that a pair $u^i$ and $u^j$ was created with the same $r$.

Before the start of the consistency check (step 3 in protocol 2 [1]):
* The sender $P_s$ knows:
    * $s_i, s_j, k_i^{s_i}, k_j^{s_j}, u^i, u^j$.
* The receiver $P_r$ knows:
    * $u^i, u^j, r^i, r^j, k_i^0, k_j^0, k_i^1, k_j^1$
    * Note: we define $r^i$ as its value as derived from $u^i$: $r^i = u^i \oplus t^i \oplus G(k_i^1)$.

The goal is for the sender $P_s$ to check that $r^i = r^j$, such that the verification passes if $r^i = r^j$ and $P_r$ is not trying to cheat, and that the verification fails with probability $\frac{1}{2}$ if  $r^i \neq r^j$ or $P_r$ is trying to cheat.

Consistency check of a pair $u^i, u^j$:
  * $P_r$ computes and sends to $P_s$:
    * $h_{i, j}^{0,0} = H(G(k_i^0) \oplus G(k_j^0))$
    * $h_{i, j}^{0,1} = H(G(k_i^0) \oplus G(k_j^1))$
    * $h_{i, j}^{1,0} = H(G(k_i^1) \oplus G(k_j^0))$
    * $h_{i, j}^{1,1} = H(G(k_i^1) \oplus G(k_j^1))$
  * $P_s$ performs following checks, fail if any of the checks fail:
    * $h_{i, j}^{s_i, s_j} = H(G(k_i^{s_i}) \oplus G(k_j^{s_j}))$
    * $h_{i, j}^{\overline{s_i}, \overline{s_j}} = H(G(k_i^{s_i}) \oplus G(k_j^{s_j}) \oplus u^i \oplus u^j)$
    * $u^i \neq u^j$
    * Note: $s_i$, $s_j$ are the bits of the random vector of $P_s$, and $\overline{s_i} = 1 - s_i$.

This check fails with probability $\frac{1}{2}$ if  $r^i \neq r^j$ or $P_r$ is trying to cheat, otherwise it passes.
To verify this we should consider various cases, here we look at one specific case and refer to the paper for other cases:
* Type 3 [1]: $r^i \neq r^j$ and two of $h_{i, j}^{a,b}$ for $a, b \in {0,1}$ are incorrect
  * wlog, assume that the adversary has set the values $h_{i, j}^{0,0}$ and $h_{i, j}^{0,1}$ correctly:
    * $h_{i, j}^{0,0} = H(G(k_i^0) \oplus G(k_j^0))$
    * $h_{i, j}^{0,1} = H(G(k_i^0) \oplus G(k_j^1))$
  * and the values for $h_{i, j}^{1,0}$ and $h_{i, j}^{1,1}$ incorrectly, a smart adversary may set these values to:
    * $h_{i, j}^{1,0} = H(G(k_i^0) \oplus G(k_j^1) \oplus u^i \oplus u^j)$
    * $h_{i, j}^{1,1} = H(G(k_i^0) \oplus G(k_j^0) \oplus u^i \oplus u^j)$
  * The verification passes if $s_i$ = 0, and the adversary learns that $s_i = 0$.
  * Because $s$ was chosen at random the verification fails with probability $\frac{1}{2}$.

Why does this work?
Consider the four cases pf $s_i$ and $s_j$:
* $s_i = 0$, $s_j = 0$, $P_s$ checks:
  * $h_{i, j}^{0, 0} = H(G(k_i^{0}) \oplus G(k_j^{0}))$
    * $h_{i, j}^{0, 0}$ was set correctly by adversary, so this check passes.
  * $h_{i, j}^{\overline{0}, \overline{0}} = H(G(k_i^{0}) \oplus G(k_j^{0}) \oplus u^i \oplus u^j)$
    * $h_{i, j}^{1, 1} = H(G(k_i^0) \oplus G(k_j^0) \oplus u^i \oplus u^j)$ (see above), thus the check passes.
  * $u^i \neq u^j$
* $s_i = 0$, $s_j = 1$, $P_s$ checks:
  * Pass, similar to case $s_i = 0$, $s_j = 0$.
* $s_i = 1$, $s_j = 0$, $P_s$ checks:
  * $h_{i, j}^{1, 0} = H(G(k_i^{1}) \oplus G(k_j^{0}))$
    * $P_r$ set $h_{i, j}^{1,0} = H(G(k_i^0) \oplus G(k_j^1) \oplus u^i \oplus u^j)$
    * Because $r^i \neq r^j$ we know that: $G(k_i^{s_i}) \oplus G(k_j^{s_j}) \oplus u^i \oplus u^j \neq G(k_i^{\overline{s_i}}) \oplus G(k_i^{\overline{s_j}})$.
    * Thus we know that $H(G(k_i^{1}) \oplus G(k_j^{0})) \neq H(G(k_i^0) \oplus G(k_j^1) \oplus u^i \oplus u^j)$ and the verification fails.
    * Note:
      * This is because $u^i \oplus u^j$
      * $= (t^i \oplus G(k_i^1) \oplus r^i) \oplus (t^j \oplus G(k_j^1) \oplus r^j)$
      * $= G(k_i^0) \oplus G(k_i^1) \oplus G(k_j^0) \oplus G(k_j^1) \oplus r^i \oplus r^j$.
      * From this it is easy to see that if $r^i = r^j$ they cancel out and: $u^i \oplus u^j = G(k_i^0) \oplus G(k_i^1) \oplus G(k_j^0) \oplus G(k_j^1)$.
      * Thus follows that iff $r^i = r^j$ then: $G(k_i^{s_i}) \oplus G(k_j^{s_j}) \oplus u^i \oplus u^j = G(k_i^{\overline{s_i}}) \oplus G(k_i^{\overline{s_j}})$.
* $s_i = 1$, $s_j = 1$, $P_s$ checks:
  * Fail, similar to case $s_i = 1$, $s_j = 0$

This way we can check that a pair of $u^i$, $u^j$ used a consistent $r$.
The protocol 2 [1] continues by performing this check for every pair of $u^i$, $u^j$.
The verification passes with probability $(\frac{1}{2})^\rho$ for $p$ cheating attempts by the adversary.
Meaning, the adversary may learn $\rho$ bits of $s$ but will be caught with a probability of $(\frac{1}{2})^\rho$.
The consistency check does not reveal any further information.

## Other Considerations
There are many details left out, so please check the reference [1].
The protocol requires further modifications to work with this consistency check.
The number of oblivious transfers has to be increased by $\rho$ the statistical security parameter.
And further, the $r$ value has to be appended with a random string $r' = r || \tau$ to prevent a malicious sender from exploiting the consistency check.
Further optimizations are possible, for example, reducing the number of consistency checks.

## References
* [1] Asharov, G., Lindell, Y., Schneider, T. and Zohner, M., 2015, April. More efficient oblivious transfer extensions with security for malicious adversaries. In Annual International Conference on the Theory and Applications of Cryptographic Techniques (pp. 673-701). Springer, Berlin, Heidelberg. [URL](https://eprint.iacr.org/2015/061)
