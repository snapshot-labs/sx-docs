# Voting Strategies

Voting strategies are the contracts used to determine the voting power (VP) of a user. Voting strategies can be created in a permissionless way however to use one, one must whitelist the strategy contract in the relevant space contract for the DAO. The most common example is using the ERC-20 token balance of a user to determine his voting power. But we could imagine other voting strategies: owning a specific NFT, owning NFT of collection X and another NFT of collection Y, having participated in protocol xyz... the possibilities are endless! 

All voting strategies must have a public view function called `get_voting_power` which gets called internally within the `propose` and `vote` functions. It has the following interface:

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
`timestamp` is the snapshot at which voting power is calculated for all users. We use a timestamp rather than a block number to better enable multi-chain applications since a timestamp is universal to all chains. There are two sets of parameters for each voting strategy, `params` and `user_params`. `params` are set in the space contract and are the same for every user who calls that strategy in the space, they act as configuration parameters for a strategy. Examples here are token contract address that will be queried, or a constant scaling factor that should be applied to the VP returned for every user. `user_params` are submitted by users when they create a proposal or cast a vote, and can therefore be different for each user. Examples here are an Ethereum storage proof. There is no requirement to use either of these parameter arrays in your strategy, in which case you can just pass an empty array. 

The voting power returned by the function is a `Uint256`, we use this type so that there is greater compatibility with Ethereum rather than using the Starknet native felt type (251 bits). When a user creates a proposal or casts a vote, the array of whitelisted voting strategies will be iterated through and `get_voting_power` will be called on each. The VP of the user will be the aggregate of the voting power from each strategy. The aggregate voting power for all users is also stored inside a `Uint256`, therefore when writing or selecting voting strategies, it is important to consider the likelihood of overflow. 

We provide the following strategies inside the sx-core repo:

### [Ethereum Balance Of](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/VotingStrategies/EthBalanceOf.cairo)

A strategy that allows balances of Ethereum ERC20 and ERC721 balances to be used as VP. 

The backend of the strategy is the Fossil module built by the awesome Oiler team. This module allows any part of the Ethereum mainnet state to be trustlessly verified on StarkNet. Verification of Ethereum state information is achieved by submitting a proof of the state to StarkNet and then verifying that proof. Once this state information has been proven, we can then calculate VP as an arbitrary function of the information. For more information on Fossil, refer to their [Github](https://github.com/OilerNetwork/fossil)

`params` should be configured as follows: 
- `params[0]`: The address of the Ethereum token contract
- `params[1]`: The index of the slot within the Ethereum token contract where the `balances[address]` mapping resides. This is required to compute the key of the storage value one is trying to prove, refer to [this](https://docs.soliditylang.org/en/v0.8.15/internals/layout_in_storage.html) section of the solidity documentation for more info. There are various tools than can be used to find this index for your storage variable such as [sol2uml](https://github.com/naddison36/sol2uml). 

`user_params` should contain the storage proof required to prove the specific value of the `balances[address]` mapping corresponding to the user. 

We built a library called [Single Slot Proof](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/lib/single_slot_proof.cairo) that interacts with the Fossil storage verifier which can be imported into your own custom voting strategies. Since **any** value of the Ethereum mainnet state can be proven here, you can get pretty creative with strategies. We are execited to see what people build! 

### [Whitelist](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/VotingStrategies/Whitelist.cairo)

A strategy that returns a specified voting power for addresses that are within a whitelist, and zero otherwise. This registers the whitelist on deployment and therefore a new instance must be deployed for each DAO that uses it. 

```
@constructor
func constructor(whitelist_len : felt, whitelist : felt*):
    register_whitelist(whitelist_len, whitelist)
    return ()
end
```

`whitelist` should be an array that has a length 3 times the number of addresses in the whitelist as we store a `Uint256` of VP for each user in addition to their addresses as follows: 
 - `whitelist[0]`: The 1st user's address
 - `whitelist[1]`: The low 128 bits of the 1st user's voting power
 - `whitelist[2]`: The high 128 bits of the 1st user's voting power
 - `whitelist[4]`: The 2nd user's address
 - etc...

Having a customizable VP for each user allows more fine grained control than enforcing a VP of 1 for everyone - which can still be achieved by just setting  the value to 1 for each member of the list. 

This strategy can be used to build a gas efficient multi-sig, which is something the Snapshot team did during the 2022 StarkNet hackathon in Amsterdam! 

### And More!

Feel free to create your own strategies! We hope that the flexibility of the system will unlock a new era of programmable on-chain governance. The interface of a strategy can be found [here](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/Interfaces/IVotingStrategy.cairo).


