# Voting strategies

Voting strategies are the contracts used to determine the voting power (VP) of a user. Voting strategies can be created in a permissionless way however to use one, one must whitelist the strategy contract in the relevant space contract for the DAO. The most common example is using the ERC-20 token balance of a user to determine his voting power. But we could imagine other voting strategies: owning a specific NFT, owning NFT of collection X and another NFT of collection Y, having participated in protocol xyz... the possibilities are endless! 

All voting strategies must have a public view function called `get_voting_power` with the following interface:

```
func get_voting_power(
    timestamp : felt,
    voter_address : Address,
    params_len : felt,
    params : felt*,
    user_params_len : felt,
    user_params : felt*,
) -> (voting_power : Uint256):
end
```
The voting power returned by the function is a `Uint256`, we use this type so that there is greater compatibility with Ethereum rather than using the Starknet native felt type (251 bits). When a user creates a proposal or casts a vote, the array of whitelisted voting strategies will be iterated through and `get_voting_power` will be called on each. The VP of the user will be the aggregate of the voting power from each strategy. 

The aggregate voting power for all users is also stored inside a `Uint256`, therefore when writing or selecting voting strategies, it is important to consider the likelihood of overflow. 



\[2] We provide the strategies:

#### [Single slot proof strategy](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/strategies/single\_slot\_proof.cairo)

Allows classic ERC20 and ERC721 balances on L1 (thanks to [Fossil](https://github.com/snapshot-labs/sx-core#Fossil-Storage-Verifier)) to be used as voting power.

#### [Vanilla strategy](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/strategies/vanilla.cairo)

Give 1 voting power to anyone, this strategy is used for testing only.

Feel free to create your own strategies! We hope that the flexibility of the system will unlock a new era of programmable on-chain governance. The interface of a strategy can be found [here](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/strategies/interface.cairo).

#### Fossil storage verifier

The backbone of the voting strategies is the Fossil module built by the awesome Oiler team. This module allows any part of the Ethereum mainnet state to be trustlessly verified on StarkNet. Verification of Ethereum state information is achieved by submitting a proof of the state to StarkNet and then verifying that proof. Once this state information has been proven, we can then calculate voting power as an arbitrary function of the information. For more information on Fossil, refer to their [Github](https://github.com/OilerNetwork/fossil)
