# Overview

Snapshot X's main contract is the **Space** contract. It handles all the book keeping needed to track proposals, votes, and results. 

A Space contract on its own won't get you very far. Indeed, the Space contract only handles the book keeping and the logic, but it does not check *who* can vote, and who has which *voting power*, or what happens when a proposal passes.

You can think about Snapshot X's architecture like creating a new character in a game. First you will need to select an armor to protect your character, then you will need to decide of a weapon (spear, sword, bow). Then you might need to choose a mount for your character to get him to travel from town to town faster.

With that analogy in mind, think about the **Space** contract like the character, and the **Authenticators**, **Voting Strategies** and **Execution Strategies** like the different add-ons you need to decide on. Let's go over those briefly:

- **Authenticators**: Determines how the participants can *authenticate* to participate in a vote. An example could be: users might need to authenticate via their metamask account. Or via an Argent X account. Or both. Or maybe you'd want your users to authenticate via their Solana wallet!
- **Voting Strategies**: Determines how the voting power is computed. The most common way of voting is voting with your coins (1 coin = 1 voting power, so if you have 100 coins you get a total voting power of 100). This is a voting strategy. But you could imagine other voting strategies: for example, only give voting power to the owners of a specific NFT. Or maybe use a quadratic model, where users only get the square root of their token balance (to minimize the power of whales). Voting strategies are just custom code that allows you to determine how you determine the voting weight of each participant.
- **Execution Strategies**: Determines what happens once a proposal passes. For example, this could be a contract that sends a message from Starknet to Ethereum, triggering some other action from another contract on Ethereum. Or this could be a sophisticated contract which deploys a contract. Or this could be the space contract itself, and the "execution" would be to change one of the configuration parameters of the space. Anything is imaginable!

Here is a diagram showcasing how the contracts that make up Snapshot X fit together along with various actions users can make to create proposals, vote, execute proposals, and update settings.

{% embed url="https://whimsical.com/snapshot-x-core-5ryHXBELMwg9L9nA6rMeDs" %}
