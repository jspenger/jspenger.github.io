---
layout: post
title: "MPC Study Group: GMW Protocol"
date: 2021-03-03
tags: GMW MPC Protocol Secure Multi-Party Computation Semi-Honest Zoom
lastupdatedon: 2021-03-03
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

# MPC Study Group: GMW Protocol
In this week's [Zoom Meetup](zoom-secure-multi-party-computation-study-group) we discussed the GMW protocol [1, 2].
You can watch the virtual meetup of the presentation followed by open discussion:

<div style="text-align: center;">
<iframe width="560" height="315" src="https://www.youtube.com/embed/QmPlq7Yofmk" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

These are my personal study notes, so please refer to the references to verify any information in this post. The content of this post is mostly based on [1].

## Introduction
The GMW protocol [2] was published in 1987, just one year after Yao's Garbled Circuits.
To my knowledge GMW was the first solution to the general secure multi-party computation problem.
The presented version here is for the secure computation of a binary circuit, however it could also be applied to arithmetic circuits [1].
More specifically, we consider the following problem:
* Secure computation of a binary circuit
* In the semi-honest setting
* With up to N-1 corruptions (N is the number of parties)

The protocol relies on the existence of a secure 1-4 [oblivious transfer](https://en.wikipedia.org/wiki/Oblivious_transfer).

Notation:
* $x$ is a plaintext value
* $[x]$ is a secret sharing of $x$
* $[x]_i$ is party $P_i$'s share of the secret

Secret sharing scheme: the protocol uses a full-threshold `t=n` secret-sharing scheme, semi-homomorphic, additive modulo 2:
* $[x]_i \in \\{0, 1\\}, x \in \\{0, 1\\}$
* $x = [x]_1 \oplus [x]_2 \oplus … \oplus [x]_n$

## The GMW Protocol
The GMW protocol consists of three steps: secret-sharing inputs; evaluating the circuit; revealing the outputs. We will look at each one of the three.

Share inputs: party $P_1$ shares input $x$.
* Party $P_i$:
  * For all $j \neq i$:
    * $[x]_j \leftarrow ^R \\{0, 1\\}$  // sample random bit
    * Send $[x]_j$ to $P_j$
  * Output $[x]_i = x \oplus  \oplus _{j \neq i}  [x]_j$

Evaluate XOR gates:
* Input $[x]$, $[y]$, output $[z]$, Party $P_i$:
  * Output $[z]_i = [x]_i \oplus [y]_i$

Evaluate NOT gates:
* Input $[x]$, output $[z]$, party $P_i$:
  * If $i = 1$: output $[z]_i = \neg [x]_i$
  * Else: output $[z]_i = [x_i]$

Evaluate AND gates:
evaluating AND gates is tricky.
For this we require interaction, whereas NOT and XOR did not require any interaction.
We use a 1-4 oblivious transfer for the interaction.
* Input $[x]$, $[y]$, output $[z = x \land y]$, Party $P_i$:
  * For all $j > i$:
    * $orecv_{i,j}$ = receive from $j$ with 1-4 oblivious transfer using selection bits: $[x]_i$, $[y]_i$
      * // author's note: $orecv_{i,j} = r_{i,j} \oplus ([x]_i \oplus [y]_j \land [x]_j \oplus [y]_i)$
  * For all $j < i$:
    * $r_{j,i} \leftarrow^R \\{0, 1\\}$  // sample random bit
    * Send to $i$ with 1-4 oblivious transfer with following input:
      * $r_{j,i} \oplus (0 \oplus [y]_j \land [x]_j \oplus 0)$
      * $r_{j,i} \oplus (0 \oplus [y]_j \land [x]_j \oplus 1)$
      * $r_{j,i} \oplus (1 \oplus [y]_j \land [x]_j \oplus 0)$
      * $r_{j,i} \oplus (1 \oplus [y]_j \land [x]_j \oplus 1)$
    * $osend_{j,i} = r_{j,i}$
  * Output $[z]_{i} = ([x]_i \land [y]_i) \oplus \oplus _{i<j} orecv _{i,j} \oplus \oplus _{i>j} osend _{j,i}$

Note that if we expand the terms of $[z]_i$ and $orecv _{i,j}$ and $osend _{j,i}$ we get:

$$[z]_i = ([x]_i \land [y]_i) \oplus \oplus _{i<j} (r _{i,j} \oplus ([x]_i \oplus  [y]_j \land [x]_j \oplus [y]_i)) \oplus \oplus _{i>j} r _{j,i}$$

Thus, if we add the shares together we get the plaintext value of $z = x \land y$:

$$
\begin{aligned}
z &=\oplus_i [z]_i \\
  &=\oplus_i \Big( ([x]_i \land [y]_i) \oplus \oplus _{i<j} (r _{i,j} \oplus ([x]_i \oplus  [y]_j \land [x]_j \oplus [y]_i)) \oplus \oplus _{i>j} r _{j,i} \Big) \\
  &=\oplus_i([x]_i \land [y]_i) \oplus \oplus _{i<j} (r _{i,j} \oplus ([x]_i \oplus  [y]_j \land [x]_j \oplus [y]_i)) \oplus \oplus _{i>j} r _{j,i} \\
  &=\oplus_i([x]_i \land [y]_i) \oplus \oplus _{i<j} ([x]_i \oplus  [y]_j \land [x]_j \oplus [y]_i) \\
  &=\oplus_i([x]_i \land [y]_i) \oplus \oplus _{i\neq j} ([x]_i \land [y]_j) \\
  &=(\oplus_i [x]_i) \land (\oplus _i [y]_i) \\
  &=x \land y
\end{aligned}
$$

The reason that the AND gate is secure, is because the receiver in 1-4 OT does not learn anything from the received values because they are masked by the random bit.
The sender also does not learn anything from the 1-4 OT because of the properties of the 1-4 OT.
Thus neither party learns about the other party's shares from the 1-4 OT execution.
The security of the protocol is inherited from the security of the 1-4 OT used and of the secret sharing scheme.
For more discussions around the security we refer to the video and to the other references [1, 2].

Reveal output: all parties reveal output $[x]$.
* Party $P_i$:
  * Announce $[x]_i$ to all other parties $j \neq i$   
  * Wait for $[x]_j$ from all $j \neq i$
  * Output $x = \oplus _i [x]_i$

## Performance
As we saw in previous section, the protocol heavily relies on 1-4 oblivious transfer.
This is the main bottleneck of the protocol and an important consideration for the performance:
* Number of 1-4 OTs per party: $O( a * (n^2 - n) / 2 )$ where $a$ is the number of AND gates.

Another performance aspect that contrasts to other protocols are the number of rounds required:
* Number of rounds: $O($ number of AND gates in the path with the most AND gates $)$.

## Questions
We discussed a few questions after the presentation:
1) Can this protocol be used for a client to outsource the computation to servers, and let the servers execute the evaluation protocol?
It seems like this should be possible with tolerance for up to n-1 failures of the n servers.
2) Could we also do a beaver triples trick in GMW, similar to BGW, in order to have less computation and communication during the evaluation phase, and how would this look? We are not sure, but it seems plausible.
3) Could we run a hybrid model between GMW and for example BGW, i.e. binary and arithmetic circuit, and what would this look like? This is a good question for future sessions.
4) What would break if we considered malicious adversaries (and assuming that the OT is maliciously secure)? It is unclear if the privacy is broken, but we assume that this may indeed be the case, and there should be caution when using the semi-honest version of GMW.

## Follow-up Suggestions
For the next week we decided to look at oblivious transfer extension, this is a way of increasing the performance of oblivious transfer.
Further, perhaps we can cover a malicious version of GMW in the future.

## References
* [1] Evans, David, Vladimir Kolesnikov, and Mike Rosulek. Section 3.2 "A pragmatic introduction to secure multi-party computation." Foundations and Trends® in Privacy and Security 2.2-3 (2017). [URL](https://securecomputation.org/)
* [2] Goldreich, Oded, Silvio Micali, and Avi Wigderson. "How to play any mental game." Proceedings of the Nineteenth ACM Symp. on Theory of Computing, STOC. ACM, 1987. [URL](https://www.cs.miami.edu/home/burt/learning/Csc609.062/docs/gmw.pdf)

## Other Helpful References
* Benny Pinkas, “Session 3: The GMW and BMR Multi-Party Protocols.” Advances in Practical Multiparty Computation, The 5th BIU Winter School, 2015. [URL](http://cyber.biu.ac.il/wp-content/uploads/2017/01/3-1.pdf) [VIDEO](https://www.youtube.com/watch?v=4YwvZaA9IEg&index=3&list=PLXF_IJaFk-9BFn8M-dsEm5x3-5Cvji3V9)
* [https://www.csa.iisc.ac.in/~arpita/SecureComputation15/Lecture11_12.pdf](https://www.csa.iisc.ac.in/~arpita/SecureComputation15/Lecture11_12.pdf)
* [https://people.eecs.berkeley.edu/~sanjamg/classes/cs276-fall14/scribe/lec16.pdf](https://people.eecs.berkeley.edu/~sanjamg/classes/cs276-fall14/scribe/lec16.pdf)
