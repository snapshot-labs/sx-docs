# Space actions

[Space](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/space/space.cairo) is the core contract: it's in charge of tracking proposals, votes, and general settings. In this section we will go into detail on how to deploy a space, and the various user actions that can take place.&#x20;

### Deploying a space

Space should be deployed via the **Proxy Factory contract**, which emits deployment events that will get indexed by the [SX-API](https://github.com/snapshot-labs/sx-api). &#x20;

To reduce deployment costs and to enable upgradability, we use ERC1967 proxies and follow Open Zeppelin's [UUPS Upgradability pattern](https://docs.openzeppelin.com/contracts/4.x/api/proxy#UUPSUpgradeable) on the space contract implementation. &#x20;

After deploying the proxy you should call the `initialize` function with the initial set of Space settings.&#x20;

```solidity
function initialize(
    address owner,
    uint32 votingDelay,
    uint32 minVotingDuration,
    uint32 maxVotingDuration,
    Strategy memory proposalValidationStrategy,
    string memory proposalValidationStrategyMetadataURI,
    string memory daoURI,
    string memory metadataURI,
    Strategy[] memory votingStrategies,
    string[] memory votingStrategyMetadataURIs,
    address[] memory authenticators
) external;
```

* `owner`: The address of the account that controls the Space contract. This account will have permissions to update space settings, cancel a proposal, and authorize an upgrade to the implementation. If you would like to remove this trust assumption, the owner can be renounced. Refer to the [Space Controller Actions](https://app.gitbook.com/o/-LFgTZvhAg63US8GVxGf/s/Z1apxjsgt60dN7Nlmu01/\~/changes/20/protocol-sx-evm/space-controller-actions/\~/comments/fcFuXklCNuVwwR4p8p3O) section for more information.
* `votingDelay`: The delay between when a proposal is created, and when the voting starts. A value of 0 means that as soon as the proposal is created anyone can vote whilst a value of 3600 means that voting will start 3600 seconds after the proposal is created.
* `minVotingDuration`: The minimum duration of the voting period. It is **impossible to execute a proposal before** **this period** has elapsed.&#x20;
* `maxVotingDuration`: The maximum duration of the voting period, it is **impossible to cast a vote after this period** has passed. The `minVotingDuration` must be less than or equal to this value.
* `proposalValidationStrategy`: A strategy that validates whether an author can create a proposal. The strategy consists of a contract address where the strategy lives, along with a set of parameters that configure it for the particular usage. More information in the [Proposal Validation Strategy](https://app.gitbook.com/o/-LFgTZvhAg63US8GVxGf/s/Z1apxjsgt60dN7Nlmu01/\~/changes/20/protocol-sx-evm/proposal-validation-strategies) section.
* `proposalValidationStrategyMetadataURI`: A metadata URI corresponding to the `proposalValidationStrategy`.
* `daoURI`: A metadata URI as defined in ERC-4824.
* `metadataURI`: The metadata URI for the space (its name, description, social tags and treasury address).
* `votingStrategies`: An array of voting strategies selected for the space. The voting power of each user will be calculated as the sum of voting powers returned for each strategy in the list for that user. More information in the [Voting Strategy](https://docs.snapshotx.xyz/protocol/voting-strategies) section.
* `votingStrategyMetadataURIs`: An array of metadata URIs corresponding to the `votingStrategies` array.&#x20;
* `authenticators`: An array of whitelisted authenticators. These are the ways in which a user can authenticate themselves in order to vote or propose. More information in the [Authenticators](https://docs.snapshotx.xyz/protocol/authenticators) section.

Each DAO on Snapshot X will have at least one space, however a DAO might choose to have multiple spaces if they want to create different rules for individual proposals. As an example one space can use an Ethereum signature as an authentication, another an Ethereum transaction.

### Creating a proposal

Once a space has been created, users can create new proposals by calling the `propose` method. Inside `propose`, the Proposal Validation Strategy is called to validate `author` upon creating a proposal.

```solidity
function propose(
    address author,
    string calldata metadataURI,
    Strategy calldata executionStrategy,
    bytes calldata userProposalValidationParams
) external;
```

* `author`: The address of the proposal author.
* `metadataURI`: The metadata URI of the proposal.
* `executionStrategy`: The execution strategy that should be used to execute a proposal if it is accepted. The strategy consists of a contract address where the strategy lives, along with an encoded execution payload. More information in the [Execution Strategies](https://app.gitbook.com/o/-LFgTZvhAg63US8GVxGf/s/Z1apxjsgt60dN7Nlmu01/\~/changes/20/protocol-sx-evm/execution-strategies) section.
* `userProposalValidationParams`: An array of parameters that will be passed to the Proposal Validation Strategy for the Space.&#x20;

Since the proposal is created via an authenticator, the exact external interface for creating a proposal will depend on the interface of the chosen authenticator.&#x20;

### Updating a proposal

Provided that the `votingDelay` has not elapsed yet, the proposal `author` can update the proposal metadata and execution strategy. This allows mistakes in the initial proposal to be fixed.

```solidity
function updateProposal(
    address author,
    uint256 proposalId,
    Strategy calldata executionStrategy,
    string calldata metadataURI
) external;
```

* `author`: The address of the proposal author.&#x20;
* `proposalId`: The unique ID of the proposal in the space. IDs are assigned incrementally to proposals based on when the proposal was created.
* `executionStrategy`: The updated execution strategy that should be used to execute a proposal if it is accepted. The strategy consists of a contract address where the strategy lives, along with an encoded execution payload. More information in the [Execution Strategies](https://app.gitbook.com/o/-LFgTZvhAg63US8GVxGf/s/Z1apxjsgt60dN7Nlmu01/\~/changes/20/protocol-sx-evm/execution-strategies) section.
* `metadataURI`: The updated metadata URI of the proposal.

Since the proposal is updated via an authenticator, the exact external interface for updating a proposal will depend on the interface of the chosen authenticator.

### Casting a vote

Once a proposal has been created, and the `votingDelay` has elapsed, users can then vote for the proposal.

```solidity
function vote(
    address voterAddress,
    uint256 proposalId,
    Choice choice,
    IndexedStrategy[] calldata userVotingStrategies,
    string calldata metadataURI
) external;
```

* `voter`: The address of the voter.
* `proposalId`: The unique ID of the proposal in the space. IDs are assigned incrementally to proposals based on when the proposal was created.
* `choice`: The vote choice: `FOR`, `AGAINST`, or `ABSTAIN`.
* `userVotingStrategies`: An array of voting strategies that should be iterated through to calculate the voter's voting power. The strategies in this array must be whitelisted by the space however there is no requirement to pass all of the whitelisted strategies. This could be useful when a user knows that they only have voting power on a subset of the whitelisted strategies and therefore can save gas by only passing strategies that they know they have non zero voting power on.&#x20;
* `metadataURI`: The metadata URI for the vote.

&#x20;Since the vote is cast via an authenticator, the exact external interface for casting a vote will depend on the interface of the chosen authenticator.&#x20;

### Execute a proposal

Calling the `execute` function on the space will call the execution strategy with the proposal state along with the execution payload. The execution strategy will then use the proposal state to compute the status of the proposal.&#x20;

If the proposal status is deemed to be `Accepted,` then the payload will be automatically executed by the strategy.&#x20;

If the proposal status is not  `Accepted` (or `VotingPeriodAccepted`), then the transaction will revert. Note that there is no caller authentication on `execute`, simply call the`execute` method on the Space contract directly:&#x20;

```solidity
function execute(uint256 proposalId, bytes calldata executionPayload) external;
```

* `proposalId`: The ID of the proposal.&#x20;
* `executionPayload`: The payload of the execution. This must be the same as the payload passed when a proposal was created. We require the payload to be resubmitted because we don't store it inside the proposal state, instead we just its hash.&#x20;

### Querying the proposal status

We provide the following view function to access the proposal status at any time.&#x20. 

```solidity
function getProposalStatus(uint256 proposalId) external view returns (ProposalStatus proposalStatus);
```

The status of a proposal is actually defined by the execution strategy chosen rather than the space itself, therefore this query actually makes an internal call to the execution strategy of the proposal. Refer to the [execution strategies](https://docs.snapshotx.xyz/protocol/execution-strategies) section for more information. 

* `proposalId`: The ID of the proposal to query.
* `voter`: The address of the voter to query.
* `proposal`: The proposal state struct object, defined [here](https://github.com/snapshot-labs/sx-evm/blob/aaed4d0dd2ad915e05fb7bad094f587fed113f7b/src/types.sol#L8).&#x20;
* `proposalStatus`: The status of a proposal at the timestamp when queried. This function will send an internal call to the `getProposalStatus` method on the execution strategy. Refer to the [Execution Strategy](https://app.gitbook.com/o/-LFgTZvhAg63US8GVxGf/s/Z1apxjsgt60dN7Nlmu01/\~/changes/20/protocol-sx-evm/execution-strategies) section for more information.

### ERC-4824: Decentralized Autonomous Organizations

The Space contract implements ERC-4824 which is a standard URI and JSON schema for DAOs, focusing on relating on-chain and off-chain representations of membership and proposals. More information can be found [here](https://ethereum-magicians.org/t/erc-4824-decentralized-autonomous-organizations/8362). The standard adds a `daoURI` string to the space settings which can be queried using the following interface: 

```solidity 
interface EIP4824 {
    function daoURI() external view returns (string _daoURI);
}
```
