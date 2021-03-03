---
layout: post
title: "MPC Study Group: BGW Protocol and Beaver Triples"
date: 2021-02-17
tags: BGW MPC Secure Multi-Party Computation Semi-Honest Zoom
lastupdatedon: 2021-02-17
---

# MPC Study Group: BGW Protocol and Beaver Triples

In this week's [Zoom Meetup](zoom-secure-multi-party-computation-study-group) we discussed the BGW protocol and Beaver triples [1, 2].
You can watch the virtual meetup of the presentation followed by open discussion:

<div style="text-align: center;">
<iframe width="560" height="315" src="https://www.youtube.com/embed/z63EYgZT054" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

These are my personal study notes, so please refer to the references to verify any information in this post. The content of this post is mostly based on [1].

## Introduction
The BGW (Ben-Or, Goldwasser, Widgerson) protocol has historical importance as one of the first secure multi-party protocols from 1988 [2].
The protocol allows us to securely compute addition-, multiplication-by-constant, and multiplication-gates over a [field](https://en.wikipedia.org/wiki/Field_(mathematics)).
The protocol relies on [secret sharing](https://en.wikipedia.org/wiki/Secret_sharing), originally for Shamir secret sharing, but the presented techniques can be applied to any linear secret-sharing scheme (LSSS).
The idea is to use the additive homomorphism property of the LSSS, such that additions and multiplication-by-constant are non-interactive operations.
However, as we will see, multiplication of two secrets still requires interaction.

## Summary
* Semi-honest security model
* Secure multi-party computation of arithmetic circuits over a field.
* Multi-party
* Security inherited from secret-sharing scheme used.
  * If using <t, n> [Shamir](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing) secret sharing, then is secure up to t corruptions. Note that for multiplication it is required that t < n/2.
  * If using [full-threshold](https://en.wikipedia.org/wiki/Secret_sharing#t_=_n) secret sharing scheme, it is secure up to n-1 corruptions.

## Notation
* `P_i` is the party with index `i`.
* `[x]` is a secret-sharing of the plain-text value `x`.
* `[x]_i` is the secret-share of `[x]` belonging to `P_i`.
* `[x_j]_i` is the secret share of `[x_j]` belonging to `P_i`, and `[x_j]` is the sharing of plain-text value `x_j`.
<!-- * Sometimes we omit writing the index if it is clear from context, for example, `[z] = [x] + [y]` could imply that every party `P_i` locally computes `[z]_i = [x]_i + [y]_i` -->

## Secret-Sharing Abstraction
The BGW protocol together with Beaver triples allows us to abstract the secret-sharing scheme [1, Sect. 3.4], such that we can use any LSSS with the following properties:
* Additive homomorphism
  * Given two shared secrets `[x], [y]`, and a publicly known value `z`, the parties can compute the following without interaction: `[x + y]`, `[x + z]`, `[x * z]`. Beaver triples (see below) are used for multiplication of two secrets.
* Opening
  * Given a shared secret `[x]`, the parties can through collaboration reveal the value `x`, i.e. reconstruct the plain-text value.
* Privacy
  * The adversary can not gain any information on the plaintext value `x` from its shares of the shared secret `[x]`.

And it should be possible to generate the following during the preprocessing phase:
* Beaver triples
  * A secret-sharing of random triples `[a], [b], [c]`, such that `c = a * b`. These are generated during the preprocessing phase, there should be one triple per multiplication gate.
* Random input gadgets
  * Random secret-sharings `[r_ij]`, for which `P_i` knows the plaintext value `r_ij`.
  There should be one per input.
  This can be used for faster secret-sharing of inputs.
  If process `P_i` has an input `x_j`, then it is sufficient to broadcast the value `d = x_j - r_ij` to all other parties, such that they each can locally compute their secret sharing as `[x] = [r_ij] + d`.

We defer the discussion around secret-sharing schemes to another time.

## Beaver Triples
Beaver triples are generated during the preprocessing phase.
There are multiple ways of generating beaver triples.
A naive way of generating the triples would to generate two random secret-sharings and multiplying them:
* Generate random shared secrets `[a] <- randomsecret()`, `[b] <- randomsecret()`, for additive and for Shamir, it is sufficient for each party to `P_i` to sample a random value as their share `[a]_i <- random()` and `[b]_i <- random()` without interaction.
* Compute `[c] = [a] * [b]` through secure multiplication protocol (there are faster methods using oblivious transfer).

Beaver triples can be used for the multiplication of two shared secrets:
* Input: Beaver triple `[a], [b], [c]`, secret-sharings `[x], [y]`. Output: `[x * y]`
  * Compute `[d] = [x] - [a]`, `[e] = [y] - [b]`
  * Reveal `d = reveal([d])` and `e = reveal([e])`.
  * All parties `P_i` compute `[x * y]_i = d*e + d*[b]_i + e*[a]_i + [c]_i`.

## BGW Protocol
Using the LSSS abstraction and beaver triples the BGW protocol becomes rather simple:
* Preprocessing (offline)
  * The parties interact to generate a sufficient amount of beaver triples and input gadgets.
* Generate Inputs
  * For each input we generate the secret-shared inputs using the input gadgets (described above).
* Compute Gates
  * Each gate in the circuit is evaluated using the abstract interface of the secret sharing (described above).
* Reveal Output
  * The output is revealed to all (or some) of the players using the abstract interface of the secret sharing.

## Discussion
We still had some open questions with regards to how to use this in practice.
For example, we are unsure on how to generate an arithmetic circuit from an arbitrary program.
Some operators like bit-shifting to the left can be achieved by multiplying by a factor of 2. Similarly, there are protocols for division.
But it still remains somewhat elusive how to implement general programs.

We also discussed issues with malicious adversaries, and decided to look at a malicious variant of the BGW protocol for next weeks session.

The question of how this, or other MPC protocols, can be used for blockchain came up, which is an interesting question for future sessions.

## Follow-up Suggestions
Malicious BGW, comparison of linear secret sharing schemes, circuit generation from programs, complexity comparison of different protocols, MPC for blockchain.

## References
* [1] Evans, David, Vladimir Kolesnikov, and Mike Rosulek. "A pragmatic introduction to secure multi-party computation." Foundations and Trends® in Privacy and Security 2, no. 2-3 (2017). URL [https://securecomputation.org](https://securecomputation.org)
* [2] Ben-Or, M., Goldwasser, S., and A. Wigderson. "Completeness Theorems for Non-cryptographic Fault-tolerant Distributed Computation." In Proceedings of the Twentieth Annual ACM Symposium on Theory of Computing (STOC'88), pp. 1-10. 1988.

## Other Helpful References
* [https://www.esat.kuleuven.be/cosic/blog/lsss/](https://www.esat.kuleuven.be/cosic/blog/lsss/)
* [https://en.wikipedia.org/wiki/Secret_sharing](https://en.wikipedia.org/wiki/Secret_sharing)
