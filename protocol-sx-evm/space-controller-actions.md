# Space controller actions

In this section we will go over the actions that can be made by the Space Controller. Note that we use Open Zeppelin's [`OwnableUpgradable`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/OwnableUpgradeable.sol) module to gate access to these functions, and therefore at the contract level we use the term `owner` instead of `controller`. &#x20;

### Cancel a proposal

A proposal can be cancelled as long as it has not already been executed.&#x20;

```solidity
function cancel(uint256 proposalId) external;
```

* `proposalId`: The ID of the proposal&#x20;

This can be used to prevent damage from malicious or broken proposals.

### Update space settings

All Space settings can be updated using the `updateSettings` function:

```solidity
function updateSettings(UpdateSettingsInput calldata input) external;

struct UpdateSettingsInput {
    uint32 minVotingDuration;
    uint32 maxVotingDuration;
    uint32 votingDelay;
    string metadataURI;
    string daoURI;
    Strategy proposalValidationStrategy;
    string proposalValidationStrategyMetadataURI;
    address[] authenticatorsToAdd;
    address[] authenticatorsToRemove;
    Strategy[] votingStrategiesToAdd;
    string[] votingStrategyMetadataURIsToAdd;
    uint8[] votingStrategiesToRemove;
}
```

A single entrypoint is used instead of separate ones for each value so that a single transaction can be used to update a large number of settings. This improves the UX while also preventing undesired behaviour that may arise if proposals are created half way through the settings update process. 

If one does not want to update a certain value, then the following placeholder values can be used in the function call (arrays can just be left empty): 

```solidity
    /// @dev Evaluates to: `0xf2cda9b13ed04e585461605c0d6e804933ca828111bd94d4e6a96c75e8b048ba`.
    bytes32 NO_UPDATE_HASH = keccak256(abi.encodePacked("No update"));

    /// @dev Evaluates to: `0xf2cda9b13ed04e585461605c0d6e804933ca8281`.
    address NO_UPDATE_ADDRESS = address(bytes20(keccak256(abi.encodePacked("No update"))));

    /// @dev Evaluates to: `0xf2cda9b1`.
    uint32 NO_UPDATE_UINT32 = uint32(bytes4(keccak256(abi.encodePacked("No update"))));
    
    Strategy NO_UPDATE_STRATEGY = Strategy(NO_UPDATE_ADDRESS, new bytes(0));
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
