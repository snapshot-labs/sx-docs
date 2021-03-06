# Voting strategies

Voting strategies are the contracts used to determine the voting power of a user. Voting strategies can be created in a permissionless way however to use one, one must whitelist the strategy contract in the relevant space contract for the DAO. The most common example is using the ERC-20 token balance of a user to determine his voting power. But we could imagine other voting strategies: owning a specific NFT, owning NFT of collection X and another NFT of collection Y, having participated in protocol xyz... the possibilities are endless! \[2] We provide the strategies:

#### [Single slot proof strategy](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/strategies/single\_slot\_proof.cairo)

Allows classic ERC20 and ERC721 balances on L1 (thanks to [Fossil](https://github.com/snapshot-labs/sx-core#Fossil-Storage-Verifier)) to be used as voting power.

#### [Vanilla strategy](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/strategies/vanilla.cairo)

Give 1 voting power to anyone, this strategy is used for testing only.

Feel free to create your own strategies! We hope that the flexibility of the system will unlock a new era of programmable on-chain governance. The interface of a strategy can be found [here](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/strategies/interface.cairo).

#### Fossil storage verifier

The backbone of the voting strategies is the Fossil module built by the awesome Oiler team. This module allows any part of the Ethereum mainnet state to be trustlessly verified on StarkNet. Verification of Ethereum state information is achieved by submitting a proof of the state to StarkNet and then verifying that proof. Once this state information has been proven, we can then calculate voting power as an arbitrary function of the information. For more information on Fossil, refer to their [Github](https://github.com/OilerNetwork/fossil)
