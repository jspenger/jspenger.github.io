---
layout: post
title: Leaderless BFT In-Place Consensus
date: 2020-06-11
tags: BFT consensus
---

# Leaderless BFT In-Place Consensus

Recently, I came across two papers: Hotstuff [1], a byzantine fault tolerant (BFT) consensus protocol, and RMWPaxos [2], an in-place consensus protocol.
This blog post is a thought experiment on: **how to combine Hotstuff and RMWPaxos to create a leaderless BFT in-place consensus protocol**.
For this, I will briefly revisit both protocols, discuss how they can be combined, followed by a short discussion.

## Hotstuff [1]
Hotstuff is a BFT consensus protocol (used by Facebook's cryptocurrency LibraBFT) under the partial synchrony system model.
The distinguishing properties of Hotstuff are:
- **Linear message complexity:** The total size of messages communicated during leader view change and value proposal are linear to the number of replicas O(n).
- **Optimistic responsiveness:** A leader must only wait for the first n-f responses to create a proposal that will make progress (leader does not wait for a timeout).

No previous protocol has simultaneous achieved both properties.
Notably, Practical Byzantine Fault Tolerance (PBFT) [3] has quadratic O(n^2) proposal complexity, and cubic O(n^3) view change message complexity, and Tendermint [4] has quadratic O(n^2) message complexity and uses timeouts (not responsive).

There are two core ideas that produce these properties:
* **Threshold signatures:** Threshold signatures are used instead of non-threshold signatures. This allows the message size of signatures to be reduced from n to 1, as n partial signatures can be combined into 1 signature, reducing the message complexity by factor n. A *quorum certificate* (QC) is a cryptographic threshold proof that at least 2f+1 replicas have signed for a value.
* **Three phases:** The consensus protocol consists of three phases instead of two phases: prepare; pre-commit; commit. In comparison, the PBFT protocol consists of two phases: prepare; commit.
The pre-commit phase guarantees that f+1 honest replicas have received and saved the QC from the prepare round.
If this QC is committed, it follows that it will be the highest QC the proposer receives during the next prepare round.
This can be achieved with linear message complexity (whilst still ensuring progress without timeouts).

Further optimisations include pipelining.
Simultaneous proposals in different stages are pipelined and grouped together.
The view is changed for every prepare phase thus each proposal has its own view.

## RMWPaxos [2]
RMWPaxos is an in-place consensus sequence protocol, assuming the crash-recovery failure model, asynchronous system model with reliable FIFO-ordered message delivery.
The protocol describes an atomic RMW (read-modify-write) register.
The distinguishing properties of RMWPaxos are:
- **In-place:** The protocol does not allocate or deallocate any variables or use logs.
- **Leaderless:** There is no elected leader that drives the proposals.
- **Single RTT for consecutive consensus decisions:** A proposer reaches consensus within 1 RTT if it is not interrupted by duelling proposers.

The single RTT for consecutive consensus decisions is achieved by acceptors automatically preparing for the next round, thus the prepare phase can be skipped.
RMWPaxos can achieve the same performance as leader-based consensus protocols (single RTT), whereas leaderless protocols such as Paxos typically require two RTTs.
The consecutive seuqence can, however, be interrupted at any point by another duelling proposer.

The core ideas behind RMWPaxos are:
- **State consensus sequences:** The proposed values are states rather than commands. A new command is applied to the current consistent state, and the resulting state is proposed as the next value in the sequence of consensus decisions. This is what allows the protocol to be in-place for consensus sequences.
- **Consistent quorums:** A quorum (majority) of replies is consistent, if the indicated state is identical for a quorum of the replies. Reading a consistent quorum is free from concurrency control. For example, a client read-command that reads from a consistent quorum can terminate early, and does not interfere with other concurrent reads and proposals. If the quorum is not consistent, then the previous consensus decision still needs to be consolidated.
- **Concurrent registers:** Because RMWPaxos is in-place and does not require logging, the protocol can efficiently be executed concurrently for multiple registers. Operations on different registers are not blocking, and a higher throughput can be achieved for many concurrent writers.

## Combined: Leaderless BFT In-Place Consensus
Is it possible to merge these ideas to create a leaderless BFT in-place consensus protocol?
Let us extend the Basic Hotstuff consensus protocol (algorithm 2 [1]) with the core ideas from RMWPaxos, to create a leaderless in-place BFT consensus protocol.

1. **In-place:**
We make the protocol in-place using state consensus sequences, by replacing *nodes* and *leafs* (tree of proposed values) with a single variable *value* (i.e. the state).
Instead of proposing a new leaf node (child of a valid node), we propose the value directly.
Because of this, no tree structures or logs must be kept.
Optionally, both values and commands can be proposed directly (for asserting that a command produces a value).

2. **Leaderless:**
We make the protocol leaderless by adding a **pre-prepare phase**.
The pre-prepare phase replaces the view-change and nextview interrupt.
The proposer sends a pre-prepare message to all acceptors.
The acceptors respond with the highest pre-committed (saved in pre-commit phase) quorum certificate (QC, the QC consists of a valid threshold signature together with the round number and value).
Furthermore, in case a proposal fails due to duelling proposals or for other reasons, a proposer may need to retry with a higher round number (or restart the pre-prepare phase), or execute a write-through of a pre-committed value that has not been committed.

3. **Two RTTs for consecutive consensus decisions:**
The pre-prepare phase can be skipped for consecutive proposals by the same proposer (assuming there are no duelling proposers), by acceptors automatically incrementing the round number in expectation that the same proposer will propose the next value.
The prepare phase can be skipped if the proposer receives a consistent quorum during the pre-prepare phase.
Reads only require a single RTT, if they read a consistent quorum, and don't have to be serialized unless they force a write-through.

4. **Concurrent consensus registers:**
Because the protocol is in-place, it is feasible to run the protocol concurrently for multiple (1000s) registers.
The proposals for disjoint (concurrent) registers do not interfere with each other and are not forced into a total-order, thus allowing for higher throughput as there is less competition.
The concurrent registers still provide linearizability.
There is a trade off between stricter ordering and throughput, some applications may require a total-order, whilst others may not.

The safety of the protocol should follow from the safety of the Basic Hotstuff protocol [1].
The liveness of the protocol is not guaranteed (liveness is impossible for the asynchronous model, see the FLP impossibility proof).

The resulting protocol has the properties: **linear message complexity**, **optimistic responsiveness** (assuming no duelling proposers), **in-place**, **leaderless**, and requires **two RTTs** in optimal path for reaching consecutive consensus decisions, and **one RTT for reads** (if reading from a consistent quroum), provides **safety** but (**no guaranteed liveness**).

## Challenges
- **Leaderless BFT consensus:** Byzantine replicas can continuously start proposing a value, potentially halting any progress from happening (no liveness). This issue could potentially be avoided by client access control over registers (e.g. byzantine replicas cannot issue commands), as each register consensus protocol can run concurrently (see for example [5, 6], a register might have consensus number 1).
- **Dynamic group membership:** Dynamic and decentralised group membership may not be possible without the use of logging. Is it possible to do in-place?

## Conclusion
Merging Hotstuff [1] and RMWPaxos [2] produces a leaderless BFT in-place consensus protocol that requires only two RTTs in the optimal path for consecutive consensus decisions for the asynchronous system model, and one single RTT in the best case for reads.
The linearizable registers can be run concurrently, which reduces proposer competition and increases throughput.
Total-order may not be necessary for BFT applications and consensus protocols, and higher throughput can be achieved if the ordering guarantees (or other guarantees) are relaxed.
Less strict ordering and delivery guarantees of BFT consensus is an interesting topic for future work [5, 6].
There remain open challenges to make the system practical, such as liveness issues and dynamic group membership.

## References
- [1] Yin, Maofan, Dahlia Malkhi, Michael K. Reiter, Guy Golan Gueta, and Ittai Abraham. "Hotstuff: Bft consensus in the lens of blockchain." arXiv preprint arXiv:1803.05069 (2018).
- [2] Skrzypczak, Jan, Florian Schintke, and Thorsten Schütt. "RMWPaxos: Fault-Tolerant In-Place Consensus Sequences." arXiv preprint arXiv:2001.03362 (2020).
- [3] Castro, Miguel, and Barbara Liskov. "Practical Byzantine fault tolerance." In OSDI, vol. 99, no. 1999, pp. 173-186. 1999.
- [4] Buchman, Ethan, Jae Kwon, and Zarko Milosevic. "The latest gossip on BFT consensus." arXiv preprint arXiv:1807.04938 (2018).
- [5] Guerraoui, Rachid, Petr Kuznetsov, Matteo Monti, Matej Pavlovič, and Dragos-Adrian Seredinschi. "The consensus number of a cryptocurrency." In Proceedings of the 2019 ACM Symposium on Principles of Distributed Computing, pp. 307-316. 2019.
- [6] Collins, Daniel, Rachid Guerraoui, Jovan Komatovic, Matteo Monti, Athanasios Xygkis, Matej Pavlovic, Petr Kuznetsov, Yvonne-Anne Pignolet, Dragos-Adrian Seredinschi, and Andrei Tonkikh. "Online Payments by Merely Broadcasting Messages (Extended Version)." arXiv preprint arXiv:2004.13184 (2020).
