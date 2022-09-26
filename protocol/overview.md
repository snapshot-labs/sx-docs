# Overview

Snapshot X's main contract is the **Space** contract. It handles all the book keeping needed to track proposals, votes, and results. 

A Space contract on its own won't get you very far. Indeed, the Space contract only handles the book keeping and the logic, but it does not check *who* can vote, and who has which *voting power*, or what happens when a proposal passes.

You can think about Snapshot X's architecture **like creating a new character in a game**. First you will need to select an armor to protect your character, then you will need to decide of a weapon (spear, sword, bow). Then you might need to choose a mount for your character to get him to travel from town to town faster.

With that analogy in mind, think about the **Space** contract like the character, and the **Authenticators**, **Voting Strategies** and **Execution Strategies** like the different add-ons you need to decide on. Let's go over those briefly:

- **Authenticators**: Determines how the participants can *authenticate* to participate in a vote. An example could be: users might need to authenticate via their metamask account. Or via an Argent X account. Or both. Or maybe you'd want your users to authenticate via their Solana wallet!
- **Voting Strategies**: Determines how the voting power is computed. The most common way of voting is voting with your coins (1 coin = 1 voting power, so if you have 100 coins you get a total voting power of 100). This is a voting strategy. But you could imagine other voting strategies: for example, only give voting power to the owners of a specific NFT. Or maybe use a quadratic model, where users only get the square root of their token balance (to minimize the power of whales). Voting strategies are just custom contracts that allow you to determine how you determine the voting power of each participant.
- **Execution Strategies**: Determines what happens once a proposal passes. For example, this could be a contract that sends a message from Starknet to Ethereum, triggering some other action from another contract on Ethereum. Or this could be a sophisticated contract which deploys a contract. Or this could be the space contract itself, and the "execution" would be to change one of the configuration parameters of the space. **Anything is possible**!

This modularity allows anyone to build with Snapshot X. Want to allow users from L1 to vote with tokens from L1? Or allow users from L2 to vote with tokens from L2? Or maybe you'd want to have a special logic where holders of a specific NFT get their voting power doubled? Anything is possible! You just need to select / write the correct **Authenticator** / **Voting Strategy** / **Execution Strategy**.

## So what does the flow look like?

Basically, the full flow looks like this:
1) A DAO creates a **Space** contract and whitelists the authenticators / voting strategies / execution strategies it's going to need.
2) A **proposal** gets created (anyone can create a new proposal as long as they have `proposer_threshold` amount of voting power (this parameter is define upon space creation and can be updated)). When created, the proposal must specify an `execution_strategy` contract along with the `execution parameters` that will get executed if quorum gets reached.
3) Users can vote (by authenticating through one of the whitelisted `Authenticator`).
4) Once the voting period has ended, anyone can call `finalize_proposal` (by interacting directly with the space contract, not with the authenticator). This will finalize the proposal, and will call the `execution strategy` defined by the proposal, forward the `execution parameters`, and specify the state of the proposal (`ACCEPTED / CANCELLED / REJECTED`).
5) The execution strategy will receive the `execution parameters` and will execute some special logic based on those (and based on whether the proposal was accepted in the first place).

Here is a diagram showcasing how the contracts that make up Snapshot X fit together along with various actions users can make to create proposals, vote, execute proposals, and update settings.

{% embed url="https://whimsical.com/snapshot-x-core-5ryHXBELMwg9L9nA6rMeDs" %}
