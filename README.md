---
description: >-
  Welcome to Snapshot X's documentation. If you're looking for specific
  technical details or just want a broad overview of the protocol, you've come
  to the right place!
---

# Snapshot X

## What's in this book?

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><h2>Protocol</h2></td><td>Technical documentation for Snapshot X protocol.<br><br>Go here if you want to understand the protocol architecture.</td><td></td><td><a href="broken-reference">Broken link</a></td></tr><tr><td><h2>User guides</h2></td><td>Non-technical documentation for the platform.</td><td><br>Go here if you want to create a space, proposal or cast a vote.</td><td><a href="broken-reference">Broken link</a></td></tr><tr><td><h2>Services</h2><p>Overview of the services built for a better UX when interacting with the protocol.<br><br>Go here if you want to integrate Snapshot X in your platform.<br></p></td><td></td><td></td><td><a href="broken-reference">Broken link</a></td></tr></tbody></table>

## What's Snapshot X?

**Snapshot X** is an on-chain voting protocol. Technically speaking, it's a set of modular smart-contracts that interact with each other to do all the book-keeping. The difference with the original [Snapshot](https://snapshot.org) is that Snapshot X is **fully on-chain**. What this means is:

* **The protocol is censorship resistant**: Anyone can cast a vote. The protocol runs without any reliance on offchain or centralized services which have the power to censor votes. \[1]
* **Voting power is computed on-chain**: The voting logic is fully on-chain and auditable so you can be sure of the logic used to compute voting power and decide on the outcome of proposals.
* **The execution is trustless**: Proposal transactions are automatically executed following the passing of a proposal. Say you create a new proposal which, if it passes, will transfer 1ETH to `vitalik.eth`. If the proposal passes, the 1ETH will automatically get sent to `vitalik.eth`, without any further human action needed.

{% hint style="info" %}
Snapshot can be found on EVM-chains and on Starknet. Both the [EVM implementation](https://github.com/snapshot-labs/sx-evm)  and the [Cairo implementation](https://github.com/snapshot-labs/sx-starknet) are open source. The Starknet-specific details can be found on [this dedicated page](protocol/starknet-specifics.md).
{% endhint %}

If anything in these docs is unclear or you would like more detail, do not hesitate to reach out on [Discord](https://discord.snapshot.org).

\[1] We note that there are in fact offchain services (eg the relayer Mana) built for use with Snapshot X, but these are not mandatory and therefore cannot lead to censorship.
