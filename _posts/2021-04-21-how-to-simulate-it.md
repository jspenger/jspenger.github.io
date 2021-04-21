---
layout: post
title: "MPC Study Group: How To Simulate It"
date: 2021-04-21
tags: How To Simulate It Simulation Proof Secure Multi-Party Computation Malicious Zoom
lastupdatedon: 2021-04-21
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

# MPC Study Group: How To Simulate It
In this week's [zoom meetup](zoom-secure-multi-party-computation-study-group) we looked at simulation based security, and how to prove MPC protocols using the simulation technique.
We watched an excellent presentation by Omer Schlomovitz  ([link](https://www.youtube.com/watch?v=_YpGx_nLJow)) to start the meetup.

These are my personal study notes, so please refer to the references to verify any information in this post.
The content of this post is mostly based on [Lindell17].

## Simulation Based Security
Simulation is a technique for reasoning about and proving the security of protocols.

Before we look at the actual definition of simulation-based security, let us consider an example.
Consider a random variable $X$, and some algorithm the simulator $S$, and a string the auxiliary information $h$.
If there exists a simulator $S$ such that the distributions $(X; h)$ and $(S(h); h)$ are indistinguishable, then no information from $X$ can be gained that cannot be gained from $h$.
In other words, if we already know $h$, then we would not know more if we also knew $X$.
This is because the simulator does not need any information about $X$ besides $h$ to generate an indistinguishable distribution to $X$.

How does this apply to cryptographic protocols?
A similar argument is used in simulation proofs, showing that no information is gained from the protocol execution that cannot be gained from the auxiliary information.
This is usually described with the real/ideal world paradigm.
In the ideal world there is an incorruptible trusted third party (TTP), which computes a cryptographic functionality with perfect security.
In the real world, there is no such TTP, but we wish to emulate the protocol to be as secure as a TTP in the ideal world.
The way to achieve this, is to show the existence of a simulator in the ideal world that can simulate the messages exchanged in the real world protocol from the point of view of the adversary.
If this is the case, in a similar argument to above, then the information gained from the real world protocol execution cannot be more than the information gained from the ideal world.
Because the ideal world does not leak any information (besides the input and output of the adversary, i.e. the auxiliary information), this implies that the real world protocol also does not leak information.
This would ensure privacy.
Another mechanism is needed to enforce correctness, by ensuring that also the outputs in the real world and ideal world match.

By proving the privacy and correctness of the real world protocol, we can say that the real world protocol securely realizes the ideal world functionality.
This has a great implication, we can use the real world protocol instance as if we were interacting with the ideal world functionality.
In order to understand the functionality, and the attacks to the functionality, it is sufficient to understand the functionality in the ideal world.
This is a subtle point, the attacks on the functionality are in the functionality description.
For example, the ideal functionality will describe whether an adversary has the power to abort the protocol or not.
This is different from the definitional approach.
In the definitional approach, we would define all attacks, and prove the protocol to be secure against this list of attacks.
Instead, with the real/ideal paradigm and simulation proofs, we list all attacks that are allowed (implicitly in the function description), and prove the protocol to be secure against all other adversaries/attacks.

I hope that there are no mistakes in my description here.
Check out the tutorial by Yehuda Lindell [Lindell17] for more (and accurate) information.

## References
* [Lindell17] Lindell, Yehuda. "How to simulate it–a tutorial on the simulation proof technique." Tutorials on the Foundations of Cryptography (2017): 277-346.
