---
layout: post
title: "MPC Study Group: GMW Compiler"
date: 2021-04-14
tags: GMW Compiler Secure Multi-Party Computation Malicious Zoom
lastupdatedon: 2021-04-14
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

# MPC Study Group: GMW Compiler
In this week's [zoom meetup](zoom-secure-multi-party-computation-study-group) we looked at the GMW compiler [GMW87; EKR17, §6.5]. We started the session by watching a video on [GMW compiler](https://www.youtube.com/watch?v=kSrTHBPLsgE) presented by Yehuda Lindell.

<div class="youtube-container">
<iframe src="https://www.youtube.com/embed/9JTA8LKsnwc" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen class="youtube-iframe"></iframe>
</div>

These are my personal study notes, so please refer to the references to verify any information in this post.
The content of this post is mostly based on [GMW87; EKR17].

## GMW Compiler
The book [ERK17] has an excellent short reference on the GMW compiler.
The goal of this blog post is to write an even shorter description of the GMW compiler.

The goal of the GMW compiler is to turn a semi-honest secure protocol into a malicious secure protocol.
The input a semi-honest protocol $\pi$ and output a new malicious secure protocol $\pi'$.
We model a probabilistic protocol as a deterministic protocol $\pi$ with explicit input of its random tape $r$ as a parameter: $\pi(r)$.
For this shorter example, we consider the case of a 2-party protocol, but it can be extended to a multi-party protocol through the use of a secure-broadcast functionality.

How does it work?
The idea of the GMW protocol is that if we have a protocol $\pi$ that we know is secure in the semi-honest model, then we can achieve a malicious secure protocol if we ensure that either every party follows the protocol specification, or abort if some party does not follow the protocol.
This can be achieved by the following mechanisms:
1. Every party commits to a random tape $r$ that is uniformly random.
2. Every party commits to its input $x$.
3. Every message sent by every party is appended a zero-knowledge proof showing that the message is consistent with an honest execution of $\pi$ on committed input $x$ and random tape $r$. If the zero-knowledge proof fails, then abort.

It should be convincing that these three rules can be applied inductively such that either every party follows the protocol specification, or the protocol is aborted (note that for the multi-party case we use a secure-broadcast functionality to ensure that every party receives the same sequence of messages).

We achieve (1) through what the book refers to as *coin-tossing into the well*.
In the 2-party case, this can be achieved by party $P_1$ committing to a random string $c = C(r)$.
Then, $P_2$ sends a plaint-text random string $r'$ to $P_1$.
$P_1$ then runs its protocol $\pi$ on the random tape $r \oplus r'$.
Because either $r$ or $r'$ is distributed uniformly random, then $r \oplus r'$ is uniformly random.
The case for $P_2$ is symmetric.

We achieve (2) through committing to the input: $c = C(x)$, with decommitment $\delta$.
The commitment is then shared with the other party.
In the actual protocol we merge (1) and (2) to create only one commitment $c$ / decommitment $\delta$ for $r$ and $x$: $c = C(x, r)$.

We achieve (3) through zero-knowledge proofs:
* Assume that it is $P_1$'s turn to send a message.
* $P_1$ computes its next message $t = \pi (x_1, r_1 \oplus r_1', T)$ which is a deterministic function on its input, random tape, and the transcript $T$ of exchanged messages.
* $P_1$ is the prover in a zero-knowledge proof with inputs $x_1, r_1, \delta_1$. The circuit is defined as:
  * $C\[\pi, c_1, r'_1, T, t\](x_1, r_1, \delta_1)$:
    * pass if $\delta_1$ is valid opening of $c_1$ to $(x_1, r_1)$, and $t = \pi(x_1, r_1 \oplus r_1', T)$. // The input and random tape is consistent, and the next message is consistent with the protocol execution on the transcript and the input.
* $P_2$ verifies the ZK proof, and aborts if verification fails.

For more information check out either [EKR17, §6.5] or the original reference [GMW87].

## References
* [GMW87] Goldreich O, Micali S, Wigderson A. How to play any mental game, or a completeness theorem for protocols with honest majority. In Providing Sound Foundations for Cryptography: On the Work of Shafi Goldwasser and Silvio Micali 2019 Oct 4 (pp. 307-328).
* [EKR17] Evans D, Kolesnikov V, Rosulek M. A pragmatic introduction to secure multi-party computation. Foundations and Trends® in Privacy and Security. 2017 Jan;2(2-3).
