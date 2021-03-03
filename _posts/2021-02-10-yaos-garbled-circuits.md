---
layout: post
title: "MPC Study Group: Yao's Garbled Circuits"
date: 2021-02-10
tags: Yao's Garbled Circuits MPC Secure Multi-Party Computation Semi-Honest Zoom
lastupdatedon: 2021-02-10
---

# MPC Study Group: Yao's Garbled Circuits
For week 1 of the [Zoom MPC Study Group](zoom-secure-multi-party-computation-study-group) we discussed Yao's Garbled Circuits, using section 3.1 from the book [A Pragmatic Introduction to Secure Multi-Party Computation](https://securecomputation.org) by David Evans, Vladimir Kolesnikov and Mike Rosulek [1].

These are my personal study notes, so please refer to the references to verify any information in this post. The content of this post is mostly based on [1].

## Introduction
Secure two-party computation was introduced in 1982 by Andrew Yao in the seminal paper "Protocols for secure computations" [2].
The work was motivated by what has come to be known as [Yao's Millionaires' Problem](https://en.wikipedia.org/wiki/Yao%27s_Millionaires%27_problem).
Two millionaires, Alice and Bob, wish to know who is richer, without revealing their respective wealth to the other.
Indeed, this would be a simple problem if either would be allowed to reveal their wealth to the other.

Yao's Garbled Circuits (YGC) [3], introduced in 1986, is a solution to the Millionaires' problem, and to general secure two-party computation problems.
In this post we will summarize our discussions from the [study group]((2021-02-03-zoom-secure-multi-party-computation-study-group)) on YGC.

## Problem Definition: Secure two-party computation
Yao's Garbled Circuits (YGC) solves: *secure two-party computation* in the *semi-honest security* model.

**What is the semi-honest security model?**
A security model is an assumption about the behaviour and power of correct parties and corrupted parties.
In the semi-honest security model, we assume that the corrupted players may not deviate from the protocol, i.e. they execute the protocol as if they were correct.
What differs between a corrupt and correct party in the semi-honest model, is that a corrupt party may try to learn information from the execution of the protocol.
This would include any information that may be extracted from the exchange of messages, such information would be for example the inputs of the other parties.

Another security model we could consider would be the *malicious security model*, in which corrupted parties may deviate from the protocol arbitrarily.

**What is secure two-party computation?**
We will define the security of secure two-party computation through the [real/ideal paradigm](https://en.wikipedia.org/wiki/Secure_multi-party_computation#Definition_and_overview).

Let us consider two parties: party `P1` and `P2`.
`P1` has some input `x1`, and `P2` has input `x2`.
Together the parties wish to calculate a function `f` with output `y <- f(x1, x2)`.

In the ideal world, there would be an incorruptible trusted entity called the ideal functionality `F` that computes the function `f`.
`P1` and `P2` would hand their inputs to the trusted non-corrupted party `F`, `F` would compute the function `f` on the inputs, and `F` would hand back the computed result to `P1` and `P2`.

It is easy to see that such an ideal functionality has the properties that we wish to emulate with MPC: privacy, i.e. the inputs are not revealed to the other parties; correctness, i.e. the function is computed correctly on the inputs; independence of inputs, i.e. no party can choose any inputs based on the inputs of other parties; fairness, i.e. all parties receive the output.

What we wish to achieve is a protocol `π` which emulates this trusted functionality.
How can we define this?
This is typically defined through simulation: an adversary may not learn any more in a real-world execution than what the adversary may learn in an ideal-world execution.
So, informally, the real-world protocol `π` is as secure as the ideal-world functionality `F`.

## Yao' Garbled Circuits: The protocol
Yao's Garbled Circuits (YGC) protocol [1, 3] consists of two steps:

**1: Garbled Circuit generation:**
Party `P1` generates the garbled circuit.
  *  *Wire label generation:*
  For each wire `wi` in the garbled circuit (this includes all input wires, output wires, and intermediate wires), `P1` generates randomly two wire labels: `wi_0 and wi_1`, where `wi_0` will signify the value `0` on the wire, and `wi_1` the value `1`.
  The wire label `wi_b` is a tuple: `wi_b = (ki_b, pi_b)`, consisting of a randomly chosen key `ki_b` and a random pointer bit `pi_b`.
  * *Garbled circuit construction:*
  For each gate in the circuit a garbled gate is generated.
  A garbled gate has two pairs of input wire labels.
  From these two pairs we construct a garbled table, consisting of four combinations of the two pairs.
  The garbled table encrypts the correct output wire label for the pair (if the inputs are `1` and `1` for an `AND` gate, the output should be the wire label for `1`, if inputs are `0` and `1` the output should be `0`, etc.), encrypted using the keys `ki_b` from respective input wire label.
  Intuitively, a pair of input wire labels can only decrypt a single row from the garbled table, thus only the correct output wire label can be decrypted from the garbled table during evaluation.
  Note that the garbled table is sorted according to the random pointer bits, otherwise the location would reveal the input plaintext values.
  * *Output decoding table:*
  Finally, a final table for decoding the output from the circuit to the corresponding plaintext values is created, similar to garbled tables but with the difference that the output is a plaintext value and not a wire label.

**2: Garbled Circuit evaluation:**
Party `P2` evaluates the garbled circuit.
  * Party `P1` sends the garbled circuit (all garbled gates) to party `P2`.
  Party `P1` chooses its inputs, and sends the corresponding wire labels of its inputs to `P2`.
  Party `P2` chooses its own inputs through an "1-out-of-2 [oblivious transfer](https://en.wikipedia.org/wiki/Oblivious_transfer)" protocol for each of `P2`'s input wires, `P1` as the sender and `P2` as the receiver.
  Oblivious transfer does not reveal the choices of `P2` to `P1`, and only reveals the chosen input wire-labels to `P2` and not the other wire-labels.
  * Party `P2` evaluates the garbled circuit. This is done by iteratively decrypting a row from each garbled gate using two input wire labels, this generates the wire labels for the remaining gates and for the output.
  * Party `P2` obtains the output using the output decoding table. Party `P2` outputs the output. Party `P2` sends the output to `P1` who also outputs the output.

The book "A Pragmatic Introduction to Secure Multi-Party Computation" [1] is a great reference, [link here](https://securecomputation.org), as is the original reference by Andrew Yao [3].

## Discussion
Why is this version of Garbled Circuits not secure against malicious adversaries?
There are many ways this protocol can be broken.
We discussed two ways:
If `P1` is corrupted, `P1` may encode the output wire-labels in the garbled gates arbitrarily.
Thus `P1` can control the output of every gate, and `P2` has no way of finding this out.
If `P2` is corrupted, then `P2` can return arbitrary output to `P1`, this would not be detected by `P1`.
The book [1] has more information on how to make it secure against malicious adversaries.

Can a garbled circuit be used multiple times?
No, if either party chooses different inputs for a subsequent round this may start leaking information about `P1`'s inputs to `P2`.
The garbled circuit has to be generated a new for each new instantiation.

What about multi-party, not just 2-party?
It is not obvious if and how this can be made into a multi-party protocol.
The BMR protocol (also found in [1]) is based on GC but works in the multi-party settings, we will cover this in future sessions.

How to make it more efficient?
There are various tricks for making YGC more efficient.
Notably, XOR gates can be evaluated for free, and the amount of ciphertexts can be halved.
For even more implementation techniques, check out [1].

How does it compare to other protocols?
We are uncertain how garbled circuits stack up against other forms of secure multi-party computation.
What we have found so far are statements that for certain problems garbled circuits may be more efficient, whereas for others arithmetic circuit protocols such as BGW [1] may be more efficient.

## Follow-up Suggestions
Things we want to cover in the future:
other circuit garbling techniques;
implementation of garbled circuits;
definition of real/ideal MPC together with simulation proofs [4].

## References
* [1] Evans, David, Vladimir Kolesnikov, and Mike Rosulek. "A pragmatic introduction to secure multi-party computation." Foundations and Trends® in Privacy and Security 2, no. 2-3 (2017). URL [https://securecomputation.org](https://securecomputation.org)
* [2] Yao, Andrew C. "Protocols for secure computations." In 23rd annual symposium on foundations of computer science (sfcs 1982), pp. 160-164. IEEE, 1982.
* [3] Andrew C. Yao, "How to generate and exchange secrets," SFCS '86 Proceedings of the 27th Annual Symposium on Foundations of Computer Science, pp. 162-167, 1986.
* [4] Lindell, Yehuda. "How to simulate it–a tutorial on the simulation proof technique." Tutorials on the Foundations of Cryptography (2017): 277-346. URL [https://eprint.iacr.org/2016/046](https://eprint.iacr.org/2016/046)

## Other Helpful References:
* [http://cyber.biu.ac.il/event/the-5th-biu-winter-school/](http://cyber.biu.ac.il/event/the-5th-biu-winter-school/)
* [https://en.wikipedia.org/wiki/Secure_multi-party_computation](https://en.wikipedia.org/wiki/Secure_multi-party_computation)
* [https://www.esat.kuleuven.be/cosic/blog/introduction-to-garbled-circuit/](https://www.esat.kuleuven.be/cosic/blog/introduction-to-garbled-circuit/)
* [https://www.youtube.com/watch?v=FTxh908u9y8](https://www.youtube.com/watch?v=FTxh908u9y8)
