---
layout: post
title: "MPC Study Group: Authenticated Garbling"
date: 2021-03-31
tags: Authenticated Garbling Secure Multi-Party Computation Malicious Zoom
lastupdatedon: 2021-03-31
---

<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {
  skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
  inlineMath: [['$','$']]
}
});
</script>
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

# MPC Study Group: Authenticated Garbling

In this week's [zoom meetup](zoom-secure-multi-party-computation-study-group) we looked at "Authenticated garbling and efficient maliciously secure two-party computation" by Wang et al. (2017) [WRK17].

<div class="youtube-container">
<iframe src="https://www.youtube.com/embed/uDEQkB1Is-k" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen class="youtube-iframe"></iframe>
</div>

These are my personal study notes, so please refer to the references to verify any information in this post.
The content of this post is mostly based on [WRK17].

## Garbled Circuits
We covered garbled circuits in our first [session](yaos-garbled-circuits) of the MPC study group.
The version that we presented was secure against semi-honest adversaries, but not against malicious adversaries.
One example to show this, the circuit generator $P_1$ may encode the wire-labels in the garbled gates arbitrarily, thus $P_1$ can control the output of the gates.
The circuit evaluator $P_2$ can return arbitrary output to $P_1$, this would not be detected by $P_1$.

In order to deal with malicious adversaries we look at authenticated garbling [WRK17].
The idea is to use authenticated secret sharing that is additively homomorphic.
Using this the parties cooperate to generate the garbled circuit.
The preprocessing phase does a lot of the heavy lifting, generating beaver triples and more that are used for the circuit generation.
Admittedly, I found this to be a tricky paper to read, and I am still working on understanding it fully.
It was very helpful to read [NNOB12] which this paper derives from.
For this overview I will instead present the more abstract form from [EKR17].

## Distributed Garbling
The garbled table of an AND gate will have the form (we'll explain what the elements mean in a moment):
* $e_{0,0} = H(k_a^0 \|\| k_b^0) \oplus k_c^{p_c \oplus p_a * p_b}$
* $e_{0,1} = H(k_a^0 \|\| k_b^1) \oplus k_c^{p_c \oplus p_a * \overline{p_b}}$
* $e_{1,0} = H(k_a^1 \|\| k_b^0) \oplus k_c^{p_c \oplus \overline{p_a} * p_b}$
* $e_{1,1} = H(k_a^1 \|\| k_b^1) \oplus k_c^{p_c \oplus \overline{p_a} * \overline{p_b}}$

How are the elements generated and what do they mean?
* Wire labels $k_i^0$, $k_i^1$:
  * $P_1$ chooses random wire labels of length $\kappa$.
* $p_a$, $p_b$, $p_c$:
  * Input pointer bits representing the value false (and $\overline{p_a}$ represents true).
  * The pointer bits are generated randomly, such that neither party knows the pointer bits.
  * The pointer bits are authenticated.
    * $[p_a]$ is the authenticated $p_a$.
    * $P_1$ (the generator) holds a uniform global key $\Delta _1$, and $P_2$ holds $\Delta _2$.
    * $P_1$ holds for an authenticated bit (the bit is not known to either $P_1$ or $P_2$): $[p_a]_1 = $
      * $<$
      * $p_{a, 1},$
      * $K_1[p_a],$ // uniform random key
      * $T = K_2[p_a] \oplus p_a \Delta _2,$ // MAC tag
      * $>$
    * Whereas, $P_2$ holds: $[p_a]_2 = $
      * $<$
      * $p_{a, 2},$
      * $K_2[p_a],$
      * $T = K_1[p_a] \oplus p_a \Delta _1,$
      * $>$
    * Such that $p_{a, 1} \oplus p_{a, 2} = p_a$  // $p_{a, i}$ is a secret-share.
    * Note that the shares are additive.
* $H$:
  * Random oracle hash function.

The above garbled table can be expanded and rewritten as:
* $e_{0,0} = H(k_a^0 \|\| k_b^0) \oplus k_c^0 \oplus (p_c \oplus p_a * p_b) \Delta $
* $e_{0,1} = H(k_a^0 \|\| k_b^1) \oplus k_c^0 \oplus (p_c \oplus p_a * \overline{p_b}) \Delta $
* $e_{1,0} = H(k_a^1 \|\| k_b^0) \oplus k_c^0 \oplus (p_c \oplus \overline{p_a} * p_b) \Delta $
* $e_{1,1} = H(k_a^1 \|\| k_b^1) \oplus k_c^0 \oplus (p_c \oplus \overline{p_a} * \overline{p_b}) \Delta $

The observation to be made is that $P_1$ knows the left side of $e_{\alpha, \beta}$:
* $H(k_a^\alpha \|\| k_b^\beta) \oplus k_c^0$

Whereas the parties hold additive shares of the right side:
* $(p_c \oplus p_a * p_b) \Delta$

Hence the garbled table is shared between the two parties.

We didn't cover the form of XOR gates, and how the garbled circuit is generated, and later evaluated.

## Performance
This paper put a lot of effort towards improving the performance.
For the function independent and function dependent pre-processing the communication and computation complexity is:
* $O(\|C\|)$

For the online phase:
* Communication complexity: $O(\|I\| + \|O\|)$
* Computation complexity: $O(\|C\|)$

## References
* [WRK17] Wang X, Ranellucci S, Katz J. Authenticated garbling and efficient maliciously secure two-party computation. In Proceedings of the 2017 ACM SIGSAC Conference on Computer and Communications Security 2017 Oct 30 (pp. 21-37). [URL](https://eprint.iacr.org/2017/030)
* [NNOB12] Nielsen JB, Nordholt PS, Orlandi C, Burra SS. A new approach to practical active-secure two-party computation. In Annual Cryptology Conference 2012 Aug 19 (pp. 681-700). Springer, Berlin, Heidelberg.
* [EKR17] Evans D, Kolesnikov V, Rosulek M. A pragmatic introduction to secure multi-party computation. Foundations and Trends® in Privacy and Security. 2017 Jan;2(2-3).
