---
layout: post
title: Using Blockchain for Tamper-Resistant Broadcast Protocols
date: 2020-05-22
tags: verifiable BFT consensus
lastupdatedon: 2020-08-24
---

# Using Blockchain for Tamper-Resistant Broadcast Protocols\*
*\*Executive Summary of: "Using Blockchain for Tamper-Proof Broadcast Protocols", submitted in partial fulfilment of the requirements for the degree of Master of Science (M. Sc.) in the Department of Computer Science at the Humboldt University of Berlin, advised by Dr. Florian Schintke, supervised by Prof. Reinefeld and Prof. Björn Scheuermann, written by Jonas Spenger, May 2020 [1].*

## 1. Introduction
This thesis was motivated by the question: *how to implement blockchain-based tamper-resistant replicated state machines (TRRSM)*? By *tamper-resistant* we refer to the distinct quality of blockchain-based applications: 1) the probabilistic (with high probability) protection against byzantine [2, sect. 2.2.6] behaviour, and 2) the probabilistic verifiability that no tampering has occurred. By *blockchain-based* we refer to implementations that use or interact with the blockchain.

### 1.1 Problem Statement
Implementing blockchain-based *tamper-resistant replicated state machines (TRRSMs)* should ideally be easy, but is currently a complex task. We experienced this in prior work implementing a Bitcoin [3] blockchain-based file system [4]. We found that the Bitcoin interface (a cryptocurrency interface with weak ordering and delivery guarantees) was difficult to use and had unsatisfactory properties for our specific use case.

In this thesis, we study a reduction of this problem: *tamper-resistant broadcast protocols (TRBPs)* using the *Bitcoin blockchain*; for various environment models and safety and liveness properties; in the asynchronous distributed system model [2, sect. 2.5.1] with byzantine failures [2, sect. 2.2.6]; assuming zero-fee Bitcoin transaction costs.

**Definition** *Tamper-Resistant Broadcast (TRB)*: The TRB abstraction extends the broadcast abstraction [2, ch. 3] by the addition of a verification protocol (event 3 and 4), and consists of four events:
* **Event 1** *Broadcast \| message*: Broadcast a message to all other processes.
* **Event 2** *Deliver \| p, message*: Deliver a message broadcast by process *p*.
* **Event 3** *Verify \| p, message*: Verify if a message has been tampered with.
* **Event 4** *VerifyReturn \| “Valid”/”NotValid”, proof*:  Return value for verification request.
* **Property 1** *Tamper-resistant*: 1) the probabilistic (with high probability) protection of the broadcast messages and the delivered message history against byzantine behaviour, and 2) the probabilistic verifiability[^1] of the delivered messages.

Consequently, the TRBPs can be used to implement TRRSMs. The environment model of the TRBP can be matched with the required environment model of the TRRSM. The safety and liveness properties of the TRBP can be matched with the required consistency model [5] of the TRRSM.

## 2. Bitcoin Blockchain (BB)
In layman’s terms, the Bitcoin blockchain (BB) [3] can be described as an append-only ledger (log) of transactions. There is disagreement on the properties of the BB in literature, however it is generally accepted as tamper-resistant.

Part of the difficulty with implementing Bitcoin blockchain-based TRRSMs stems from Bitcoin’s probabilistic nature, this includes that appended transactions are not final and may be removed from the ledger, and so-called forks (temporary inconsistencies of the tail-end of transaction histories).

## 3. Tamper-Resistant Broadcast (TRB)
The presented tamper-resistant broadcast protocols use the BB to disseminate messages, by writing messages as data-entries [6] into BB transactions. The transactions are added to the BB ledger, and propagated via the BB network to other participating instances of the TRBPs. The TR property of the TRB is inherited from the BB (i.e., if BB is TR, then TRB is TR). Besides the TR property, we looked at how to achieve other safety and liveness properties, under different environment models.

Let us consider an example to clarify how we can implement a broadcast primitive using the BB, and how we can extend its properties. The Best-Effort [2, sect. 3.2] Tamper-Resistant Broadcast (BEBTRB) primitive has four characteristic properties: *tamper-resistant*; *validity*; *no-duplication*; *no-creation*. Our BETRB protocol creates a valid transaction from the message the client requests to broadcast, and continually re-appends this transaction to the BB (validity, follows from BB’s fair-append property). The BB ledger is continually checked for changes, and any transactions in the BB ledger that have not been delivered before (no-duplication) are delivered to the client. Transactions are always signed by the sending BETRB instance (no-creation, follows from BB’s transaction validity property). The tamper-resistant property is inherited from the use of BB.

We looked at three different environment models: public, permissioned, and non-native permissioned.

The *public environment* (protocol is open to anyone, participants may be unknown) is suitable for public protocols, for which *causal-order* and *reliable delivery* suffice (total-order and consensus is not possible in the public environment [7]).

The *permissioned environment* (participants are known) is suitable for protocols that require *total-order* and *uniform reliable delivery* (and group membership). We also note that tamper-proof is achievable in the permissioned setting, but not in the public.

The *non-native permissioned environment model* (messages are broadcast via direct links, but not via blockchain) is suitable for protocols that require *high performance* (throughput, latency). We implemented a non-native tamper-resistant broadcast protocol, and our evaluation [8] confirmed a significant performance improvement, throughput by 20 times, latency by 10,000 times, compared to the non-native counterpart.

## 4. Tamper-Resistant Replicated State Machine (TRRSM)
A ​replicated state machine is a program which is replicated on several processes [2, sect. 6.9]. We can implement a TRRSM using a TRB instance by: TRB broadcasting any incoming commands, executing TRB delivered commands on the state machine, and saving the TRB delivered commands to a log. The TRRSM inherits the TR property from the TRB.

We show how to implement four blockchain technology use cases with the TRB [9]: a file system; a timestamp server; a virtual blockchain; and a supply chain.

## 5. Related Work
Our definition of tamper-resistant extends probabilistic [2, sect. 1.5] byzantine fault tolerance [2, sect. 2.2.6] with an explicit verifiability protocol (which otherwise is implicit).

There are many examples of blockchain-based applications. In particular, our use cases show how to implement abstractions of: Blockstack’s VirtualChain [10] virtual blockchain (public); Factom’s [11] and Chainpoint’s [12] timestamp servers (non-native permissioned); Endolith’s [13] file system (permissioned). These are specific instances of the problem, our work encompasses these and more instances, and we tried to include the various solutions from literature in our protocols.

## 5. Conclusion
Building blockchain-based applications is difficult, and requires complex interaction with the blockchain. We have shown that the blockchain-based tamper-resistant broadcast is an expressive abstraction which solves this problem, confirmed by the use cases and by prior work [4].

We recommend further work on the possibilities and limitations of tamper-resistant and tamper-proof applications, increasing the performance of systems, exploring the space of properties and environments, for example, a convincing case can be made for a reliable and causal-order blockchain for the public environment, or for a reliable byzantine broadcast [14] for decentralized online payments.

## References
- [1] Jonas Spenger. Using Blockchain for Tamper-Proof Broadcast Protocols. 2020. URL [http://nbn-resolving.de/urn:nbn:de:0297-zib-79165](http://nbn-resolving.de/urn:nbn:de:0297-zib-79165).
- [2] Christian Cachin, Rachid Guerraoui, and Luis Rodrigues. Introduction to reliable and secure distributed programming. Springer Science & Business Media, 2011.
- [3] Satoshi Nakamoto. Bitcoin: A peer-to-peer electronic cash system, 2008.
- [4] Pacio – audit-proof, distributed archival service, Last visited 2019-05-13. URL [https://www.zib.de/projects/pacio](https://www.zib.de/projects/pacio).
- [5] Paolo Viotti and Marko Vukolic. Consistency in non-transactional distributed storage systems. ACM Computing Surveys (CSUR), 49(1):1–34, 2016.
- [6] Andrew Sward, Ivy Vecna, and Forrest Stonedahl. Data insertion in bitcoin’s blockchain. Ledger, 3, 2018.
- [7] Kenji Saito and Hiroyuki Yamada. What’s so different about blockchain? — blockchain is a probabilistic state machine. In 2016 IEEE 36th International Conference on Distributed Computing Systems Workshops (ICDCSW), pages 168–175. IEEE, 2016.
- [8] Source code, Last visited 2020-04-07. URL [https://github.com/jonasspenger/mscthesis](https://github.com/jonasspenger/mscthesis). Used commit short hash 1db6931.
- [9] Fran Casino, Thomas K Dasaklis, and Constantinos Patsakis. A systematic literature review of blockchain-based applications: current status, classification and open issues. Telematics and Informatics, 36:55–81, 2019.
- [10] Muneeb Ali, Jude Nelson, Ryan Shea, and Michael J Freedman. Blockstack: A global naming and storage system secured by blockchains. In 2016 USENIX Annual Technical Conference, pages 181–194, 2016.
- [11] Factom - Business Processes Secured by Immutable Audit Trails on the Blockchain, Last visited 2020-01-17. URL https://www.factom.com/assets/docs/Factom_Whitepaper_v1.2.pdf
- [12] Wayne Vaughan, Jason Bukowski, Shawn Wilkinson. Chainpoint - a scalable protocol for anchoring data in the blockchain and generating blockchain receipts v2.1.0, 2016, Last visited 2020-01-17. URL https://github.com/chainpoint/whitepaper/blob/master/chainpoint_white_paper.pdf.
- [13] Thomas Renner, Johannes Müller, and Odej Kao. Endolith: A blockchain-based framework to enhance data retention in cloud storages. In 2018 26th Euromicro International Conference on Parallel, Distributed and Network-based Processing (PDP), pages 627–634. IEEE, 2018.
- [14] Daniel Collins, Rachid Guerraoui, Jovan Komatovic, Matteo Monti, Athanasios Xygkis, Matej Pavlovic, Petr Kuznetsov, Yvonne-Anne Pignolet, Dragos-Adrian Seredinschi, and Andrei Tonkikh. Online Payments by Merely Broadcasting Messages (Extended Version). arXiv preprint arXiv:2004.13184 (2020).

## Notes
[^1]: A verification request returns ”Valid” (with high probability), if the message has not been tampered with and is part of the untampered message history, else ”NotValid”.
