# Voting strategies

Voting strategies are the contracts used to **determine the voting power (VP) of users**. Voting strategies can be created in a **permissionless** way, however to use one, one must whitelist the strategy in the space contract for your DAO.

The most common example is using the ERC-20 token balance of a user to determine their voting power. But you could imagine other voting strategies: owning a specific NFT, owning NFT of collection X and another NFT of collection Y, having participated in protocol xyz... the possibilities are endless! We are fans of the [Turing Complete Governance](https://baby.mirror.xyz/O7a922A-9zT4C4UwssRExkftdHywJ-13sR2rxQ-t\_\_k), the concept of governance system with arbitrary programmability. This will allow complex and expressive mechanisms of coordination to be seemlessly integrated into governance decisions.&#x20;

Snapshot X reaches this standard, and we are excited to see what people build!

**All voting strategies must have the following** [**interface**](https://github.com/snapshot-labs/sx-evm/blob/main/src/interfaces/IVotingStrategy.sol)**:**&#x20;

```solidity
function getVotingPower(
    uint32 timestamp,
    address voterAddress,
    bytes calldata params,
    bytes calldata userParams
) external returns (uint256);
```

`getVotingPower` is called internally by the space contract when a vote is cast. `timestamp` is the snapshot at which voting power is calculated for all users. We use a **timestamp rather than a block number** to better enable multi-chain applications since a timestamp is universal to all chains.

There are two sets of parameters for each voting strategy, `params` and `userParams`.

* `params` are set in the space contract and are the same for every user who calls that strategy in the space, they act as configuration parameters for a strategy. Examples here are token contract address that will be queried, or a constant scaling factor that should be applied to the VP returned for every user.
* `userParams` are submitted by users when they create a proposal or cast a vote, and can therefore be different for each user. Example here is a storage proof.

There is no requirement to use either of these parameter arrays in your strategy, in which case you can just pass an empty array.

When a user casts a vote, the array of whitelisted voting strategies will be iterated through and `getVotingPower` will be called on each. The VP of the user will be the aggregate of the voting power from each strategy. The aggregate voting power for all users is also stored inside a `Uint256`, therefore when writing or selecting voting strategies, it is **important to consider the likelihood of overflow**.

DAOs are free to write their own custom strategies that suit their own needs however we provide the following approaches:

### [Compound Style Checkpoint Token](https://github.com/snapshot-labs/sx-evm/blob/main/src/voting-strategies/CompVotingStrategy.sol)

A strategy that allows delegated balances of Compound style checkpoint tokens to be used as voting power. To use this strategy, your token must have the following interface exposed:&#x20;

```solidity
function getPriorVotes(address account, uint256 blockNumber) external view returns (uint96);
```

When calling this strategy, `params` should be:

```solidity
params = abi.encodePacked(tokenAddress)
```

`userParams` is not needed. &#x20;

### [OpenZeppelin Style Checkpoint Token](https://github.com/snapshot-labs/sx-evm/blob/main/src/voting-strategies/OZVotesVotingStrategy.sol)

A strategy that allows delegated balances of OpenZeppelin style checkpoint tokens to be used as voting power. To use this strategy, your token must have the following interface exposed:&#x20;

```solidity
function getPastVotes(address account, uint256 blockNumber) external view returns (uint256);
```

When calling this strategy, `params` should be:

```solidity
params = abi.encodePacked(tokenAddress)
```

`userParams` is not needed.  &#x20;

### [Whitelist](https://github.com/snapshot-labs/sx-evm/blob/main/src/voting-strategies/WhitelistStrategy.sol)

A strategy that returns a specified voting power for addresses that are within a whitelist, and zero otherwise. Having a customisable VP for each user allows more fine grained control than enforcing a VP of 1 for everyone - which can still be achieved by just setting the value to 1 for each member of the list.  Each member of the whitelist should be represented as the struct:

```solidity
struct Member {
    address addr;
    uint256 vp;
}
```

When calling this strategy, `params` should then be an ABI encoded array of type `Member[]` where the members are sorted into ascending order based on their `addr`.

<pre class="language-solidity"><code class="lang-solidity"><strong>Member[] members = ...
</strong><strong>params = abi.encode(members)
</strong></code></pre>

`userParams` is not needed.  &#x20;

### And more!

Feel free to create your own strategies!&#x20;

We hope that the flexibility of the system will unlock a new era of programmable on-chain governance. The interface of a strategy can be found [here](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/Interfaces/IVotingStrategy.cairo).
