---
layout: post
title: "MPC Study Group: SPDZ Malicious BGW MPC Protocol"
date: 2021-02-24
tags: SPDZ Malicious BGW MPC Secure Multi-Party Computation Zoom
lastupdatedon: 2021-02-24
---

# MPC Study Group: SPDZ Malicious BGW MPC Protocol

In this week's [Zoom MPC Study Group](zoom-secure-multi-party-computation-study-group) we read about the SPDZ protocol, a malicious-secure BGW variant [1, 2].
You can watch the virtual meetup of the presentation followed by the open discussion:

<div class="youtube-container">
<iframe src="https://www.youtube.com/embed/hUrBS8TloFM" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen class="youtube-iframe"></iframe>
</div>

These are my personal study notes, so please refer to the references to verify any information in this post. The content of this post is mostly based on [1, Section 6.6].

## Introduction
Why is the vanilla [BGW](bgw-protocol-and-beaver-triples)  protocol not secure against malicious adversaries?
In the previous week's session we only considered semi-honest secure sharings.
If we instead consider a malicious-secure sharing, that has certain properties, we will find that the BGW protocol is maliciously secure.

The BGW protocol is malicious-secure if we have a sharing mechanism with the following properties [1] (compare this to the properties from the secret sharing abstraction in [BGW](bgw-protocol-and-beaver-triples), differences highlighted in bold):
* Additive homomorphism:
  * Given two shared secrets `[x]`, `[y]`, and a publicly known value `z`, the parties can compute the following without interaction: `[x + y]`, `[x + z]`, `[x * z]`.
* Privacy:
  * A **malicious** adversary can not gain any information on the plaintext value from its shares of the shared secret.
* Opening:
  * Given a shared secret `[x]`, the parties can through collaboration reveal the value `x`, i.e. reconstruct the plain-text value, even in the presence of **malicious** adversaries.

We will look at the SPDZ protocol and how it achieves these properties.
The book [1, Section 6.6] is a good reference for a quick overview, but the original paper is required to get all the details involved [2].

## Summary
* Malicious security model, [UC-secure](https://en.wikipedia.org/wiki/Universal_composability).
* Secure multi-party computation of arithmetic circuits over a field.
* Multi-party.

## SPDZ
The core idea of SPDZ is to use a secret-shared MAC key `[∆]` to authenticate the shares, the key is not known to any party.
Using it the parties can hold both the secret-shared value `[x]`, and an authentication share `[∆ * x]`, both of which are additive.

Notation:
* Party `P_i`, and the set of all parties `{P_1, ..., P_n}`.
* `x` is a plaintext value, `[x]` is an additive secret-sharing of `x` (i.e. `x = [x]_1 + ... [x]_n`), `[x]_i` is party `P_i`'s share.
* `∆` is the global MAC key, and `[∆]_i` is party `P_1`'s secret share of the key.
* `<x> = (d, [x],  [∆ * x])` is an authenticated secret sharing of `[x]`, and `P_i`'s share `<x>_i`.

The protocol has three steps: first we need to generate the authenticated shares; then we can compute the arithmetic circuit on the shares like for BGW; and lastly we reveal and authenticate that the revealed share has not been corrupted.

Share generation: this step is very detailed, so we defer to the original paper for its description [2].
In this step the authenticated shares of the inputs, and authenticated beaver triples are generated.

Evaluate arithmetic circuit: the parties evaluate every gate of the circuit.
* Inputs:
  * `<x> = (d_x, [x],  [∆ * x])`, authenticated share
  * `<y> = (d_y, [y],  [∆ * y])`, authenticated share
  * `w`, plaintext value
  * `<a>, <b>, <c>`, authenticated beaver triple
* Addition of two shares (non-interactive): `<x>`, `<y>`
  * `<z> =  (d_x + d_y, [x + y]  , [∆ * x  +  ∆ * y]) =  (d_x + d_y, [x] + [y], [∆ * x] + [∆ * y])`
* Addition of a share with a plaintext value (non-interactive): `<x>`, `w`
  * `<z>_1 = (d_x - w, [x]_1 + w, [∆ * x]_1)`, for player 1 non-interactive operations.
  * `<z>_i = (d_x - w, [x], [∆ * x])`, for all other parties `i != 1`.
* Multiplication of a share with a plaintext value (non-interactive): `<x> = (d_x, [x],  [∆ * x])`, `w`
  * `<z> = (d_x * w, [x] * w, [∆ * x] * w)`
* Multiplication of two shares using the beaver trick (assuming that the beaver triple is correct) (interactive): `<x>`, `<y>`
  * Compute `<f> = <x> - <a>`, `<e> = <y> - <b>`
  * Reveal (without authenticating / without revealing the MAC key) `f` and `e` to all parties.
  * `<x * y> = <c> + e*<b> + f*<a> + e*f`

Authenticate and reveal: the checking of the shares authenticity is delayed until the end of the protocol.
This means that shares may be corrupted early on in the evaluation, but we do not check for it, and amy not be aware of it.
The reason why it has to be delayed is that we need to reveal the MAC before we can check the authenticity of any shares, but once the MAC has been revealed an adversary can forge an authentication for other corrupted shares, because the same global MAC key is used for all shares. The following description differs from [2] and resembles more [1] for a single output.
* Inputs `<x>` and `[∆]_i`
* Reveal `<x>`
  * All parties `P_i` announce their shares `[x]_i`, and compute `x = [x]_1 + ... + [x]_n`.
  * All parties `P_i` commit to their value `F_com( [∆]_i*x-[∆*x]_i )`, and share this commit with all other parties.
  * All parties `P_i` open the commits of the other parties, such that every party has for each `j` the committed value `[∆]_j*x-[∆*x]_j`.
  * All parties check the sum of the committed values: `check 0 == ([∆]_1*x-[∆*x]_1) + ... + ([∆]_n*x-[∆*x]_n)`.
    * Abort if the sum is not `0`.
  * Output `x`

Disclaimer: I likely have missed some essential steps, so please check out the original paper [2].

## Discussion
Could SPDZ be adapted to work with other secret sharings, for example Shamir secret sharing?

A malicious party can abort the protocol at any time, and there is no output fairness.
Would it be possible to find the identity of the party that has aborted?

## Follow-up Suggestions
Other malicious-secure secret sharings; complexity comparison of MPC protocols; MPC with identifiable abort.

## References
* [1] Evans, David, Vladimir Kolesnikov, and Mike Rosulek. "A pragmatic introduction to secure multi-party computation." Foundations and Trends® in Privacy and Security 2.2-3 (2017). [URL](https://securecomputation.org/)
* [2] Damgård, Ivan, Valerio Pastro, Nigel Smart, and Sarah Zakarias. "Multiparty computation from somewhat homomorphic encryption." In Annual Cryptology Conference, pp. 643-662. Springer, Berlin, Heidelberg, 2012. [URL](https://eprint.iacr.org/2011/535)
