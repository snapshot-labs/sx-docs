# Space controller actions

In this section we will go over the actions that can be made by the Space Controller. Note that we use Open Zeppelin's [`OwnableUpgradable`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/OwnableUpgradeable.sol) module to gate access to this functions, and therefore at the contract level we use the term `owner` instead of `controller`. &#x20;

### Cancel a proposal

A proposal can be cancelled as long as it has not already been executed.&#x20;

```solidity
function cancel(uint256 proposalId) external;
```

* `proposalId`: The ID of the proposal&#x20;

This can be used to prevent damage from malicious or broken proposals.

### Update space settings

All Space settings can be updated using the following functions:

```solidity
function setVotingDelay(uint32 delay) external;

function setMinVotingDuration(uint32 duration) external;

function setMaxVotingDuration(uint32 duration) external;

function setProposalValidationStrategy(Strategy calldata proposalValidationStrategy) external;

function setMetadataURI(string calldata metadataURI) external;

function addVotingStrategies(
    Strategy[] calldata votingStrategies,
    string[] calldata votingStrategyMetadataURIs
) external;

function removeVotingStrategies(uint8[] calldata indicesToRemove) external;

function addAuthenticators(address[] calldata _authenticators) external;

function removeAuthenticators(address[] calldata _authenticators) external;
```

{% hint style="warning" %}
Note that Space setting updates will not affect the functioning of ongoing proposals at the time of the settings update since we store the necessary settings data inside the state of each proposal.&#x20;
{% endhint %}

### Upgrade implementation

A Space contract's implementation can be upgraded to a new version using the following methods: &#x20;

```solidity
function upgradeTo(address newImplementation) external; 

function upgradeToAndCall(address newImplementation, bytes memory data) external;
```

* `newImplementation`: The address of the new space implementation.&#x20;
* `data`: A encoded function call to initialize the new implementation.&#x20;

Refer to [Open Zeppelin's UUPS guide](https://docs.openzeppelin.com/contracts/4.x/api/proxy#UUPSUpgradeable) for more information.&#x20;

### Manage ownership&#x20;

```solidity
function transferOwnership(address newOwner) external;

function renounceOwnership() external; 
```

The `owner` can be transferred to a new address or renounced entirely.&#x20;
