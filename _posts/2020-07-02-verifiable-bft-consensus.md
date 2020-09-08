---
layout: post
title: Verifiable BFT Consensus
date: 2020-07-02
tags: verifiable BFT consensus
lastupdatedon: 2020-09-08
---

# Verifiable Byzantine Fault Tolerant Consensus

The byzantine fault tolerant (BFT) consensus abstraction [1, 2] ensures the consistency among correct (non-faulty) instances even if some instances are byzantine (faulty, incorrect).
The correct instances all decide on the same consensus value, whereas byzantine instances may exhibit arbitrary behaviour.

This abstraction has been proven to be useful.
However, the abstraction in its simplicity leaves out important functionality for real-world systems.
One issue of the BFT consensus abstraction is that a third-party client cannot distinguish a correct instance from a byzantine instance, and thus cannot distinguish a correct consensus decision from a faulty consensus decision.

We address this issue by answering the question: *how can a consensus decision be trusted?*. We define the *verifiable BFT consensus* abstraction with added functionality to make consensus decisions verifiable, as inspired by the verifiability of real-world blockchain implementations such as Bitcoin and Ethereum.

## Problem Statement
Byzantine instances cannot be distinguished from correct (non-byzantine) instances in deployments when the client and server (consensus instance) do not share the same failure domain.
Because of this, there is a need for functionality to verify the validity of a consensus decision.

The verifiable byzantine fault tolerant consensus abstraction extends the BFT consensus abstraction with such functionality.
In the next sections we will discuss how this can be implemented and how it relates to other work such as Bitcoin and Ethereum.

**Definition** *Verifiable Byzantine Fault Tolerant Consensus (VBFTC)*: The VBFT consensus abstraction extends the BFT consensus abstraction from [2], the additions are underlined:
- **Event 1** *Propose \| v*: Proposes value v for consensus.
- **Event 2** *Decide \| v, <ins>proof</ins>*: Outputs a decided value *v* of consensus <ins>together with a verifiable *proof*</ins>.
- **Property 1-4** *Termination, integrity, validity and agreement*: Same as BFT consensus [2].
- <ins>**Property 5** *Verifiability*: If a decided value *v* and provided *proof* (correct or byzantine) is verified by a verification function *ver(v, proof)* as valid, then every correct process eventually decides on *v*. The output of a correct process is always verified as true.</ins>

Consequently, a client could safely interact with any instance of the VBFT consensus, byzantine or non-byzantine, without risk of being manipulated by a byzantine instance.
If a consensus decision is verified as valid, then it is safe to assume that every correct process eventually decides on the same value.

## Implementing VBFTC
There are many possible ways of implementing VBFTC, we consider two naive methods of transforming a non-verifiable BFTC to a verified BFTC for this:
- **Method 1** The VBFTC instance simulates a (non-verifiable) BFTC instance, and outputs together with the consensus decision from the BFTC instance the history of all incoming and outgoing messages of the BFTC instance. The message history should be a sufficient proof. This method requires knowledge of how the BFTC protocol works because it is simulated.
- **Method 2** The VBFTC instance is connected to every BFTC instance, and waits for consensus decision messages. If it receives f+1 consensus decisions with the same output (f is the highest number of allowed failures), then the VBFT instance can safely assume the value to be correct. The decided value can be output together with the proof consisting of the f+1 messages.

We note that there is no difference in power between BFTC and VBFTC, as BFTC can be transformed into VBFTC and vice versa.

## Related Work
The Practical Byzantine Fault Tolerance (PBFT) protocol [3] implicitly implements *method 2*.
In the final step of the protocol the client waits for confirmation from at least f+1 instances, this is the proof that the output is correct and not faulty.

We are unsure if the Hotstuff protocol [4] implements verifiability similarly to PBFT, as it is reported that each instance 'responds to clients' at the end of the decision phase.
This may be similar to PBFT.
Alternatively, implementing verifiability into the Hotstuff protocol could be achieved by returning the 'quorum certificate' as a valid proof.
The quorum certificate is a threshold signature.
A threshold signature signature requires signatures from at least 2f+1 different instance, and is constant in size and runtime.

The Bitcoin blockchain [5] has functionality to verify that a transaction has been added to the blockchain through the functions *verifychain* and *gettxoutproof* *verifytxoutproof*, and we classify it as using *method 1*.
However, complex interaction with the Bitcoin interface is required for a thorough check of the validity.
Furthermore, the durability of Bitcoin proofs are suspicious, as transactions and their corresponding proofs may be invalidated due to Bitcoin forks (at a diminishing probability).
It may be more suitable to term Bitcoin a *probabilistically verifiable BFTC* protocol.

## Discussion
Every output is verifiable of the VBFTC abstraction.
We believe this is useful for situations in which the client and server (consensus instance) belong to different failure domains, which applies to web applications and *edge computing* applications.
As a consequence, the abstraction enables us to fully decouple the application from the consensus (blockchain), and allows us to form a many-to-many relationship between consensus instances and application instances, whilst providing application-to-application verifiable security.

Dealing with dynamic group membership may be challenging.
By dynamic group membership we mean that the participating consensus instances may change over time.
A group is also called a view, and a group change is called a view change.
A proof would need to prove that the current view is correct and that the current view decided on the value.
Proving that the current view is correct may require proving that the previous view was correct, and that the previous view decided on the current view, this would cause the proof size to grow linearly with the number of view changes and instances.
This is a challenge for practical implementations.

Assuming the existence of threshold cryptography, we found that we can generate proofs in O(f) time-, message-, and space complexity, and verify proofs in constant O(1) time-, message, and space complexity, with proofs being constant O(1) in size.
We are unsure if a constant proof size is possible for dynamic group membership, although it might be feasible under an amortised model.
If threshold cryptography and authentication cannot be assumed, then we would require O(f) in proof size and complexity.

There may be environments for which it is impossible to achieve verifiability.
The public unbounded environment (number of instances is unknown, e.g. Bitcoin), implies that consensus cannot be solved [6], which in turn implies that verifiability cannot be solved.
This can be circumvented by relaxation of the properties.
In the public unbounded environment we can achieve probabilistic consensus and probabilistic verifiability, i.e. the consensus and verification protocol are correct with a high probability.
This relaxation can further enable improvements on the complexity (c.f. [7]).

We suspect that it is easy to get an implementation of VBFTC wrong, compromising the security.
It may be a good idea to check the correctness of other BFT consensus implementations with respect to their (implicit or explicit) verification protocols.

## Conclusions
Verifiability is useful, but easy to get wrong.
Challenges to make it practical include reducing the time-, message-, and space complexity, especially for dynamic group membership.
A possibility is to further explore probabilistic verifiability and probabilistic consensus for reducing the complexity, or by assuming static group membership.

## References
- [1] Shostak, Robert, Marshall Pease, and Leslie Lamport. "The byzantine generals problem." ACM Transactions on Programming Languages and Systems 4, no. 3 (1982): 382-401.
- [2] Cachin, Christian, Rachid Guerraoui, and Luís Rodrigues. "Byzantine Consensus." Introduction to reliable and secure distributed programming, 244-261. Springer Science & Business Media, 2011.
- [3] Castro, Miguel, and Barbara Liskov. "Practical Byzantine fault tolerance." In OSDI, vol. 99, no. 1999, pp. 173-186. 1999.
- [4] Yin, Maofan, Dahlia Malkhi, Michael K. Reiter, Guy Golan Gueta, and Ittai Abraham. "Hotstuff: Bft consensus in the lens of blockchain." arXiv preprint arXiv:1803.05069 (2018).
- [5] Nakamoto, Satoshi. "Bitcoin: A peer-to-peer electronic cash system." (2008).
- [6] Saito, Kenji, and Hiroyuki Yamada. "What’s so different about blockchain?—Blockchain is a probabilistic state machine." In 2016 IEEE 36th International Conference on Distributed Computing Systems Workshops (ICDCSW), pp. 168-175. IEEE, 2016.
- [7] Collins, Daniel, Rachid Guerraoui, Jovan Komatovic, Matteo Monti, Athanasios Xygkis, Matej Pavlovic, Petr Kuznetsov, Yvonne-Anne Pignolet, Dragos-Adrian Seredinschi, and Andrei Tonkikh. "Online Payments by Merely Broadcasting Messages (Extended Version)." arXiv preprint arXiv:2004.13184 (2020).

## Update
Since writing this post I have found other articles on this topic that may be worth reading:
- Setty, Srinath, Sebastian Angel, Trinabh Gupta, and Jonathan Lee. "Proving the correct execution of concurrent services in zero-knowledge." In 13th USENIX Symposium on Operating Systems Design and Implementation (OSDI 18), pp. 339-356. 2018.
- Setty, Srinath, Sebastian Angel, and Jonathan Lee. "Verifiable state machines: Proofs that untrusted services operate correctly." ACM SIGOPS Operating Systems Review 54, no. 1 (2020): 40-46.
