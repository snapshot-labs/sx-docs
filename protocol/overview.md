# Overview

Snapshot X's main contract is the **Space** contract. It handles all the book keeping needed to track proposals, votes, and results.

A Space contract on its own won't get you very far. Indeed, the Space contract only handles the book keeping and the logic, but it does not check _who_ can vote, and who has which _voting power_, or what happens when a proposal passes.

You can think about Snapshot X's architecture **like creating a new character in a game**. First you will need to select an armor to protect your character, then you will need to decide of a weapon (spear, sword, bow). Then you might need to choose a mount for your character to get him to travel from town to town faster.

With that analogy in mind, think about the **Space** contract like the character, and the **Authenticators**, **Voting Strategies** and **Execution Strategies** like the different add-ons you need to decide on. Let's go over those briefly:

* **Authenticators**: Determines how the participants can _authenticate_ to participate in a vote. An example could be: users might need to authenticate via their Ethereum Metamask account. Or via an StarkNet Argent X account. Or both. Or maybe you'd want your users to authenticate via their Solana wallet!
* **Voting Strategies**: Determines how the voting power is computed. The most common way of voting is voting with your tokens (1 token = 1 voting power, so if you have 100 tokens you get a total voting power of 100). This is a voting strategy. But you could imagine other voting strategies: for example, only give voting power to the owners of a specific NFT. Or maybe use a quadratic model, where users only get the square root of their token balance (to minimize the power of whales). Voting strategies are just custom contracts that allow you to determine how you determine the voting power of each participant. Arbitrary logic can be implemented here to create complex and expressive governance mechanisms.
* **Execution Strategies**: Determines what happens once a proposal passes. For example, this could be a contract that sends a message from Starknet to Ethereum, triggering some other action from another contract on Ethereum. Or this could be a sophisticated contract which deploys a contract. Or this could be the space contract itself, and the "execution" would be to change one of the configuration parameters of the space. **Anything is possible**!

This modularity allows anyone to build with Snapshot X. Want to allow users from L1 to vote with tokens from L1? Or allow users from L2 to vote with tokens from L2? Or maybe you'd want to have a special logic where only holders of a specific NFT can create new proposals? Anything is possible! You just need to select / write the correct **Authenticator** / **Voting Strategy** / **Execution Strategy**.

## So what does the flow look like?

Basically, the full flow looks like this:

1. A DAO creates a **Space** contract and whitelists the authenticators / voting strategies / execution strategies it's going to need.
2. A user can create a **proposal** in the **Space** via an authenticator as long as they exceed a threshold amount of voting power with respect to the voting strategies specified. When created, the proposal must specify an execution strategy contract along with a set of execution parameters that contain transactions to be executed if a proposal passes.
3. Users can vote in the proposal by authenticating through one of the whitelisted authenticator contracts.
4. Once the voting period has ended, anyone can finalize the proposal. This will update the state of the proposal and will call the execution strategy with the execution parameters and the proposal outcome (`ACCEPTED / CANCELLED / REJECTED`).
5. The execution strategy will then execute logic using the proposal outcome and parameters supplied.

Here is a diagram showcasing how the contracts that make up Snapshot X fit together along with various actions users can make to create proposals, vote, execute proposals, and update settings.

{% embed url="https://whimsical.com/snapshot-x-core-5ryHXBELMwg9L9nA6rMeDs" %}
