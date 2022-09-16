## Deploying a Space

The [space](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/space/space.cairo) is THE core contract: it's in charge of tracking proposals, votes, and other general settings. To deploy a new space, you will need to provide:

* `voting_delay`: The delay between when a proposal is created, and when the voting starts. A value of 0 means that as soon as the proposal is created anyone can vote whilst a value of 3600 means that voting will start 3600 seconds after the proposal is created.
* `min_voting_duration`: The minimum duration of the voting period, it is impossible to finalize the proposal before this period is over, even if the quorum for thw proposal is met.
* `max_voting_duration`: The maximum duration of the voting period, it is impossible to cast a vote after this period has passed. Obviously, the `min_voting_duration` must be less than the `max_voting_duration`.
* `proposal_threshold`: The minimum amount of voting power needed to be able to create a new proposal in the space.
* `controller`: The address of the controller account for a space. This account will have permissions to update space settings and cancel a proposal. This clearly introduces a trust assumption, a way to remove this is to make the controller another space and therefore settings updates or proposal cancellations must be voted on by the community.  
* `quorum`: The minimum total voting power required for a proposal to pass. Note this is the combined voting power of all votes, not just `FOR` votes. 
* `voting_strategies`: An array of voting strategy contracts addresses that define the whitelisted voting strategies used by the space. The voting power of each user will be calculated as the sum of voting powers returned for each strategy in the list for that user. More information in the [Voting Strategy](https://github.com/snapshot-labs/sx-core#Voting-Strategies) section.
* `voting_strategy_params_flat`: An array of parameters for the voting strategies specified in `voting_strategies`. This would be a 2D array but Cairo cannot curently handle that data type in calldata therefore we must pass a flattened version. 
* `authenticators`: An array of whitelisted authenticators. These are the ways in which a user can authenticate themselves in order to vote or propose. For more information, refer to the [Authenticators](https://github.com/snapshot-labs/sx-core#Authenticators) section.
* `executors`: An array of accepted execution strategies. These strategies will handle the execution of transactions inside proposals once voting is complete. More information about execution in the [Execution Contract](https://github.com/snapshot-labs/sx-core#Execution-Contract) section.

Spaces should be deployed via a Space Factory contract using the `deploy_space` method. The factory tracks deployment events which get indexed by the SX-API. Each DAO on Snapshot X will have at least one space, however a DAO might choose to have multiple spaces if they want to create different 'categories' of proposal each with different settings.

## Creating a Proposal 

Once a space has been created, users can create new proposals by calling the `propose` method (provided the caller has at least `proposal_threshold` voting power). Users don't directly interact with the `space` contract, but instead via one of the whitelisted `authenticator` contracts which act as proxies. Since the proposal is created via an authenticator, the exact external interface for creating a proposal will depend on the interface of the chosen authenticator. The parameters passed from the authenticator to the space contract will be the following: 

* `proposer_address`: The address of the proposal creator. Snapshot X is designed to for multi-chain environments so we do not enforce any particular address type. For Ethereum DAOs, this would be an Ethereum address.
* `metadata_uri`: Pointer to the location of the metadata for the proposal that the proposal creator should upload.
* `executor`: The execution strategy that should used after the proposal is complete.
* `execution_params`: Parameters required by the execution strategy. Often this will include a hash of the transactions inside a proposal.
* `used_voting_strategies`: An array of voting strategies that should be iterated through to calculate the proposer's voting power. This must exceed `proposal_threshold` in order for the proposal to be created. The strategy addresses in this array must be whitelisted by the space however there is no requirement to pass all of the whitelisted strategies. This could be useful when a user knows that they only have voting power on a subset of the whitelisted strategies and therefore can save gas by only passing strategies that they know they have non zero voting power on.
* `user_voting_strategy_params_flat`: The user parameters required by the voting strategies specified in `used_voting_strategies`. This would be a 2D array but Cairo cannot curently handle that data type in calldata therefore we must pass a flattened version. 

## Casting a Vote 

Once a proposal has been created, and the `voting_delay` has elapsed, users can then vote for the proposal (once again, using an `authenticator` as a proxy). Since the vote is cast via an authenticator, the exact external interface for casting a vote will depend on the interface of the chosen authenticator. The parameters passed from the authenticator to the space contract will be the following: 

* `voter_address`: The address of the voter. Snapshot X is designed to for multi-chain environments so we do not enforce any particular address type. For Ethereum DAOs, this would be an Ethereum address.
* `proposal_id`: The unique ID of the proposal in the space. IDs are assigned incrementally to proposals based on when the proposal was created.
* `choice`: The vote choice; `FOR`, `AGAINST`, or `ABSTAIN`.
* `used_voting_strategies`: An array of voting strategies that should be iterated through to calculate the voter's voting power. The strategy addresses in this array must be whitelisted by the space however there is no requirement to pass all of the whitelisted strategies. This could be useful when a user knows that they only have voting power on a subset of the whitelisted strategies and therefore can save gas by only passing strategies that they know they have non zero voting power on.
* `user_voting_strategy_params_flat`: The user parameters required by the voting strategies specified in `used_voting_strategies`. This would be a 2D array but Cairo cannot curently handle that data type in calldata therefore we must pass a flattened version. 

## Executing the Proposal

Once the `min_voting_duration` has passed, `finalize_proposal` can be called provided that the `quorum` for the space has been reached. The caller of `finalize_proposal` must pass the `proposal_id` along with the same `execution_params` as when the proposal was created. The `executor` contract will be called with the `execution_params` to execute the proposal. Note that there is no caller authentication required for `finalize_proposal`, transactions can be permissionlessly submitted directly to the space contract provided the conditions above are met. 

```
@external
func finalize_proposal(proposal_id : felt, execution_params_len : felt, execution_params : felt*):
```

## Cancelling the Proposal 

We provide a method for the `controller` account to cancel a proposal in case of an issue with it. The arguments passed are the same as for `finalize_proposal`. Certain execution strategies might have particular logic to execute in case of a proposal transaction, so the execution payload is still forwarded to the relevant `executor` contract.

```
@external
func cancel_proposal{syscall_ptr : felt*, pedersen_ptr : HashBuiltin*, range_check_ptr : felt}(
    proposal_id : felt, execution_params_len : felt, execution_params : felt*
):
```

