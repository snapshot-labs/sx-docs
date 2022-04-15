# Space

The [space](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/space/space.cairo) is THE core contract: it's in charge of tracking proposals, votes, and other general settings. To deploy a new space, you will need to provide:

* `voting_delay`: The delay between when a proposal is created, and when the voting starts. A `voting_delay` of 0 means that as soon as the proposal is created, anyone can vote. A `voting_delay` of 3600 means that voting will start `3600` seconds after the proposal is created.
* `voting_duration`: The duration of the voting period, i.e. how long people will be able to vote for a proposal.
* `proposal_threshold`: The minimum amount of voting power needed to be able to create a new proposal in the space. Used to avoid having anyone be able to create a proposal.
* `voting_strategies`: A list of voting strategy contracts addresses that define the voting strategies used by the space. The voting power of each user will be calculated as the sum of voting powers returned for each strategy in the list for that user. More information in the [Voting Strategy](https://github.com/snapshot-labs/sx-core#Voting-Strategies) section.
* `authenticators`: A list of accepted authenticators. These are the ways in which a user can authenticate themselves in order to vote or propose. For more information, refer to the [Authenticators](https://github.com/snapshot-labs/sx-core#Authenticators) section.
* `executor`: The execution strategy contract that will handle the execution of transactions inside proposals once voting is complete. More information about execution in the [Execution Contract](https://github.com/snapshot-labs/sx-core#Execution-Contract) section.

Once a space has been created, users can create new proposals by calling the `propose` method (provided the caller has at least `proposal_threshold` voting power). Users don't directly interact with the `space` contract, but instead via one of the `authenticator` contracts that act as proxies.

The proposal creator will need to provide the following parameters:

* `proposer_address`: The Ethereum address of the proposal creator which will be used to check that their voting power exceeds the `proposal_threshold`.
* `metadata_uri`: Pointer to the location of the metadata for the proposal that the proposal creator should upload.
* `execution_hash`: Hash of all of the transactions inside the proposal.
* `execution_params`: Additional parameters required by the execution strategy.
* `ethereum_block_number`: The Ethereum block number at which point the snapshot of voting power is taken.
* `voting_strategy_params`: The parameters required by the voting strategies used by the space.

Once a proposal has been created, and the `voting_delay` has elapsed, users can then vote for the proposal (once again, using an `authenticator` as a proxy).

Voters will need provide the following parameters:

* `proposal_id`: The ID of the proposal in the space they want to vote in.
* `voter_address`: The Ethereum address of the proposal creator which will be used to calculate their voting power.
* `choice`: The votes choice; `FOR`, `AGAINST`, or `ABSTAIN`.
* `voting_strategy_params`: The parameters required by the voting strategies used by the space.

Once the `voting_duration` has passed, votes are closed, and anyone can call `finalize_proposal` (this time directly on the space contract as no authentication is required): this will finalize the proposal, count the votes for/against/abstain, and call the `executor`.

Note that each DAO will have at least one space, however a DAO might choose to have multiple spaces if they want to create different 'categories' of proposal each with different settings.

### Space factory

To enable an easy way to deploy and keep track of spaces, each DAO will have a space factory contract that will do this. The factory pattern has not yet been released on StarkNet therefore we are waiting to implement this feature.
