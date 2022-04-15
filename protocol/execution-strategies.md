# Execution strategies

The `executor` contract implements the execution strategy that gets called when voting for a proposal is done. The interface can be found [here](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/execution/interface.cairo). Once voting has ended, calling `finalize_proposal` in the space contract will pass the following parameters to the `execute` method of the `executor`:

* `proposal_outcome`: Whether the proposal has passed or not (majority of voting power allocated to `FOR` signifies that the proposal has passed).
* `execution_hash`: Hash of the transactions to be executed.
* `execution_params`: Array of additional parameters that are needed by the execution strategy.

Currently this repo provides the Zodiac Execution Strategy. This enables a DAO to perform permissionless execution of L1 Gnosis Safe transactions upon the successful completion of a proposal. The [Zodiac Relayer](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/execution/zodiac\_relayer.cairo%60) contract is the `executor` for this strategy, which will forward the execution to the [L1 Zodiac module](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/ethereum/SnapshotXZodiacModule/SnapshotXL1Executor.sol) address specified in `executions_params[0]`. To use this strategy, DAOs will need a Gnosis Safe with the Snapshot X Zodiac module activated.

We will also be adding a StarkNet transaction execution strategy in the near future.
