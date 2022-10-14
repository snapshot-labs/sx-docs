# Snapshot X

Welcome to Snapshot X's documentation! If you're looking for specific technical details or just want a broad overview of the protocol, you've come to the right place!

# What's Snapshot X?

**Snapshot X** is a voting tool for DAOs. Simple as that! Technically speaking, it's a set of highly modular smart-contracts that interact with each other to do all the book-keeping, but the end result is the same: it's a **voting tool**!

The difference between Snapshot X and [Snapshot](https://snapshot.org) is that Snapshot X is **fully on-chain**. What this means is:

- **Votes are permissionless**: Anyone can cast a vote. The protocol runs without any reliance on offchain or centralized services which have the power to censor votes. [1]
- **Votes are trustless**: The voting logic is fully on-chain and auditable so you can be sure of the logic used to compute voting power and decide on the outcome of proposals.
- **Execution is trustless**: Proposal transactions are automtically executed following the passing of a proposal. Say you create a new proposal which, if it passes, will transfer 1ETH to `vitalik.eth`. If the proposal passes, the 1ETH will automatically get sent to `vitalik.eth`, without any further human action needed.

# What's in this book?

- Protocol : Documentation on the Snapshot X protocol and how to extend it to fit your usecase. 
- Services : Documentation on the various offchain services that were built to enhance the UX of interacting with the protocol. 

If anything in these docs is unclear or you would like more detail, do not hesitate to reach out us on [discord](https://discord.gg/snapshot) :)

[1] We note that there are in fact offchain services (Mana, UI) built for use with Snapshot X, but these are not mandatory and therefore cannot lead to censorship. 
