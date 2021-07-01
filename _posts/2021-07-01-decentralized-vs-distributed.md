---
layout: post
title: The Differences and Similarities between Distributed and Decentralized Systems
date: 2021-07-01
---

Many discussions in the field of distributed systems often describe blockchain systems as 'distributed' and 'decentralized' computer systems. At the same time, these discussions. The goal of this blog post is to elaborate on the notion of _decentralization_ and _distribution_ in the context of computer systems, and to highlight the differences and similarities between systems exhibiting these properties.

## Decentralized vs. Distributed Systems

The discussion on decentralized computer systems goes back a long time. Already in 1962, Paul Baran elaborated on the differences between centralized, decentralized, and distributed systems in the context of communications networks with the following figure:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ site.baseurl }}/assets/img/decentralized_vs_distributed_baran.png">
    </div>
</div>
<div class="caption">
    The differences between centralized, decentralized, and distributed networks (Paul Baran, 1962).
</div>

While this figure is often used by contemporary articles to shed light on the differences between decentralized and distributed systems, it falsely raises the impression that these properties are the result of the underlying network topology. Even though decentralized networks often avoid a centralized point of communication, the definition of decentralization (and distribution) require a more rigorous definition.

### The Principles of Decentralized Systems

Decentralization is mostly about _authority_, resolving around the question on _who_ can do _what_ in the system. As such, decentralization not only concerns technical aspects of the system but also relates to the authority of end users to influence the behaviour of the system. A system where all authority resides within a single entity is often referred to as _centralized_. Most of the digital systems that we are using nowadays are centralized, for example, Facebook, Twitter and Google. Even though these platforms provide us with free services, it is ultimately the company behind these services making the decision on what you will see, and how you can interact with others.

Centralized solutions are prone to manipulation by the operating authority. Therefore, various systems attempt to delegate the decision making away from single authorities to end users instead. We refer to systems where (a part of) the decisions in the network are made by end users as _decentralized_.

Other systems can have semi-decentralized aspects. For example, these systems could be under the control of a group of users with different affiliations. Permissioned blockchains are an example of this, where a group of pre-approved nodes maintain the integrity of the blockchain ledger (e.g., a consortium of companies). Compared to centralized systems, semi-decentralized systems are often less vulnerable to malicious behaviour since this would often require the collusion between multiple nodes.

We provide a few examples of decentralized systems below:
- Localized decisions.

As might become apparent from the examples above. A system is often classified as centralized or decentralized based on a few components. However, it can be argued that _a system in itself_ cannot be considered as centralized or decentralized, because these systems consists of many components, and there can be large differences in decentralization between components. [A recent article](https://www.sciencedirect.com/science/article/pii/S0306457321000844) identifies 13 points of centralization in the Bitcoin ecosystem only, including the division of assets between different wallets, the concentration of mining power, and the geographical distribution of nodes in the network. This shows that the discussion around decentralization is not trivial.

From a technical perspective, decentralization is not always easy to achieve.
- law of redecentralization

### The Principles of Distributed Systems

TODO

(Algorand as counter-example?)