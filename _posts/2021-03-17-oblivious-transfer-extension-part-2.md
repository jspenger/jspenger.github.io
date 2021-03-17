---
layout: post
title: "MPC Study Group: Oblivious Transfer Extension Part 2"
date: 2021-03-17
tags: Oblivious Transfer Extension Secure Multi-Party Computation Semi-Honest Zoom
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

# MPC Study Group: Oblivious Transfer Extension Part 2

In this week's [zoom meetup](zoom-secure-multi-party-computation-study-group) we looked at "More Efficient Oblivious Transfer Extensions with Security for Malicious Adversaries" by Asharov et al. (2015) [1].
We got stuck on the description of protocol 1 [1], which is the semi-honest secure OT extension protocol of Ishai et al. (2003) [2] (per notation from [3]) that we discussed last week, albeit with some optimizations.
What we found was that the notation differed from what we discussed in [last week's session](oblivious-transfer-extension).

These are some personal study notes to clear up some of the confusion between the description from [1] and [last week](oblivious-transfer-extension).

## Differences and Similarities
The two protocols are shown in figures 1 and 2, in this section we refer to protocol 1 from [1] as fig. 1, and fig. 2 for [2, 3].

<p align="center">
  <img src="/assets/images/1-2-ot-extension-part-2-asharov.png" width="100%"/>
  <br/>
  <em>Figure 1: OT Extension, Protocol 1 from Asharov et al. (2015) [1]. Index $i$ is used for denoting columns, index $j$ for rows.</em>
</p>

<p align="center">
  <img src="/assets/images/1-2-ot-extension-part-2-lw.png" width="100%"/>
  <br/>
  <em>Figure 2: OT Extension, Ishai et al. (2003)[2], per notation from [3]. Index subscript $_i$ is used for denoting rows, index superscript $^i$ is used for denoting columns.</em>
</p>


There are some immediately noticeable differences:
* Figure 1 [1] uses a pseudorandom generator (PRG): $G$, whereas figure 2 [2, 3] doesn't.
* In figure 1 (1a), the receiver $R$ chooses random $k_i^0, k_i^1$ of length $k$, whereas in figure 2 in step (2) $R$ chooses $t_j, u_j$ of length $m$ randomly.
* Further, in fig. 1 the receiver $R$ transmits columns $k_i^0, k_i^1$ via the OT, whereas in fig. 2 $R$ transmits the columns $t^j, u^j$.
  This is somewhat confusing, as $k_i^0, k_i^1$ does not directly correspond to $t^j, u^j$.
  Further, in (2 a) of fig. 1 we define $t^j, u^j$ using $k_i^0, k_i^1$, and this definition differs from fig. 2.

The protocols may seem very different, but when we take a detailed look at $t^i, u^i$, $q^i$,  etc. in both protocols we notice that the protocols are related.
* How are $T = t^0 \| ... \| t^k$ and $U = u^0 \| ... \| u^k$ related?
  * Fig. 1:
    * $t^i \oplus u^j = G(k_i^0) \oplus G(k_i^0) \oplus G(k_i^1) \oplus r = G(k_i^1) \oplus r$
  * Fig. 2:
    * $t_i \oplus u_i = \\{r_i\\}^k$, from which we can infer:
    * $t^i \oplus u^i =r$
  * We can see that $t^i \oplus u^i$ differ by $\oplus G(k_i^1)$ for fig. 1 and fig. 2.
* How are $q^i$ related?
  * Fig. 1:
    * $q^i = (\\{s_i\\}^k * u^i) \oplus G( k_i^{s_i})$  // as defined
    * $= (\\{s_i\\}^k * (G( k_i^0) \oplus G( k_i^1) \oplus r)) \oplus G( k_i^{s_i})$ // replace $u^i$.
    * If $s_i = 0$ then: $q^i =  G( k_i^0)$.
    * If $s_i = 1$ then: $q^i =  (G( k_i^0) \oplus G( k_i^1) \oplus r) \oplus G( k_i^1 ) = G( k_i^0) \oplus r$
    * And if we replace $G(k_i^0)$ with $t^i$ we get:
    * If $s_i = 0$ then: $q^i = t^i$.
    * If $s_i = 1$ then: $q^i = t^i \oplus r$.
  * Fig. 2:
    * If $s_j = 0$ then: $q^j = t^j$.
    * If $s_j = 1$ then: $q^j = u^j = t^j \oplus r$.
    * Another way of writing this is as: $q^i = (\\{s_i\\}^k * t^i) \oplus t^i$.
  * $q^j$ is the same for both fig. 1 and fig. 2.
* How are $q_j$ related?
  * $q_j$ is the same for both fig. 1 and fig. 2, this follows directly from previous step $q^i$.
  * Fig. 1 and fig. 2:
    * $q_j = (\\{r_j\\}^k * s) \oplus t_j$.

At this point we should be convinced that the protocols are related.

## Correctness of Protocol 1
To better understand the notation of Asharov et al. (2015), let us reason why the output from the protocol is correct in fig. 1 (and leaving security for a later discussion).

To show the correctness of the output, it should suffice to show that for any $j$ the output is $x_j = x^{r_j}_j$.
Let us try the case when $r_j = 0$ (case $r_j = 1$ is similar):
* In (2 e): $x_j = y_j^{r_j} \oplus H(j, t_j)$
* $x_j = y_j^0 \oplus H(j, t_j)$  // replacing $r_j$ with $0$
* $x_j = x_j^0 \oplus H(j, q_j) \oplus H(j, t_j)$  // replacing $y_j^0$
* $x_j = x_j^0 \oplus H(j, (\\{r_j\\}^k * s) \oplus t_j) \oplus H(j, t_j)$  // replacing $q_j = (\\{r_j\\}^k * s) \oplus t_j$
* $x_j = x_j^0 \oplus H(j, (\\{r_j\\}^k * s) \oplus t_j) \oplus H(j, t_j)$  // replacing $r_j$ with $0$, case assumption
* $x_j = x_j^0 \oplus H(j, (\\{0\\}^k * s) \oplus t_j) \oplus H(j, t_j)$
* $x_j = x_j^0 \oplus H(j, \oplus t_j) \oplus H(j, t_j)$
* $x_j = x_j^0$  // qed

## Why is Protocol 1 Insecure Against Malicious Adversaries?
Protocol 1 is secure for a malicious sender and semi-honest receiver [1].
But it is not clear why it is insecure for a malicious receiver.
Although it is mentioned in [1] that a malicious receiver $R$ may choose different $r$ when computing $u^i$ in step (2 a) fig. 1, it is not elaborated any further how such an attack could work.
Luckily this attack is described in Ishai et al. (2003) [2].
What we present here is an adapted version of the attack for our notation.
The malicious receiver changes the $j'$th bit of $r$ from 0 to 1, for the $i'$th computation of $u^i$.
Together with the assumption that the receiver knows the of the $j'$th input-pair, the receiver can learn the $i'$th bit of $s$:
* Let us assume that the malicious receiver $R$ has knowledge of an input pair $(x_{j'}^0, x_{j'}^1)$ for a $j' \in \\{0, ..., m\\}$, and wlog $r_{j'} = 0$, row $j \in \\{0, ..., m\\}$, column $i \in \\{0, ..., k\\}$.
* Note that $R$ has knowledge of $t_i'$ from the protocol execution.
* In (2 a):
  * For all $i \neq i': u^i = t^i \oplus G(k_i^1) \oplus r$
  * Note that $r$ is $0$ at index $j'$, i.e. $r_{j'} = 0$
  * For $i': u^{i'} = t^{i'} \oplus G(k_{i'}^1) \oplus r'$, where $r'$ is $r$ but with a $1$ at index $j'$:
    * For all $j \neq j': r'_j = r_j$  
    * For $j'$: $r'_{j'} = 1$
* In (2 d):
  * Note that:
    * For all $j \neq j': q_j = (\\{r_j\\}^k * s) \oplus t_j$
    * For $j'$:
      * $q_{j'} = ( \hat{r} * s) \oplus t_{j'}$
      * where $\hat{r} = [0, ..., 0, 1, 0, ..., 0]$ is 0 at all indices except at index $i'$ where $\hat{r}_{i'} = 1$.
      * $q_{j'} \oplus s = ( \hat{r} * s) \oplus t_{j'} \oplus s$
      * $= ( [0, ..., 0, 1, 0, ..., 0] * s) \oplus t_{j'} \oplus s$
      * $= [0, ..., 0, s_{i'}, 0, ..., 0] \oplus t_{j'} \oplus s$
      * $= [s_1, ..., s_{i'-1}, 0, s_{i'+1}, ..., s_{k}] \oplus t_{j'}$
  * If $x_{j'}^1 = y_{j'}^1 \oplus H(j', t_{j'})$ then **output** $s_{j'} = 0$.
  * If $x_{j'}^1 = y_{j'}^1 \oplus H(j', t_{j'} \oplus [0, ..., 0, 1, 0, ..., 0])$ then **output** $s_{j'} = 1$.
  * Note:
    * Where $[0, ..., 0, 1, 0, ..., 0]$ is the vector of zeroes except at index $j'$.
    * Note that the sender sets $y_{j'}^1 =  x_{j'}^1 \oplus H(j', q_{j'} \oplus s)$
    * Correctness argument:
      * If $s_{i'} = 0$ then:
        * $y_{j'}^1 \oplus H(j', t_{j'})$
        * $= x_{j'}^1 \oplus H(j', q_{j'} \oplus s) \oplus H(j', t_{j'})$
        * $= x_{j'}^1 \oplus H(j', [s_1, ..., s_{i'-1}, 0, s_{i'+1}, ..., s_{k}] \oplus t_{j'} \oplus s) \oplus H(j', t_{j'})$  // replace $ q_{j'}$
        * $= x_{j'}^1 \oplus H(j', [0, ..., 0, s_{i'}, 0, ..., 0] \oplus t_{j'} ) \oplus H(j', t_{j'})$
        * $= x_{j'}^1 \oplus H(j', [0, ..., 0, 0, 0, ..., 0] \oplus t_{j'} ) \oplus H(j', t_{j'})$  // because $s_{i'} = 0$
        * $= x_{j'}^1 \oplus H(j', \oplus t_{j'} ) \oplus H(j', t_{j'})$
        * $= x_{j'}^1$ qed.
      * Similar argument for $s_{i'} = 1$


## Follow-up
There were other questions that were unanswered: 1) why is it sufficient to ensure the consistency of $r$ for the malicious security model;
2) how does the malicious security protocol in [1] work.

## References
* [1] Asharov, G., Lindell, Y., Schneider, T. and Zohner, M., 2015, April. More efficient oblivious transfer extensions with security for malicious adversaries. In Annual International Conference on the Theory and Applications of Cryptographic Techniques (pp. 673-701). Springer, Berlin, Heidelberg. [URL](https://eprint.iacr.org/2015/061)
* [2] Ishai, Y., J. Kilian, K. Nissim, and E. Petrank. 2003. “Extending Oblivious Transfers Efficiently”. In:Advances in Cryptology – CRYPTO 2003. Ed.by D. Boneh. Vol. 2729.Lecture Notes in Computer Science. Springer,Heidelberg. 145–161.
* [3] Evans, David, Vladimir Kolesnikov, and Mike Rosulek. Section 3.7 "A pragmatic introduction to secure multi-party computation." Foundations and Trends® in Privacy and Security 2.2-3 (2017). [URL](https://securecomputation.org/)
