# Execution Strategies

Execution strategies (also interchangably called executors) are contracts that perform the logic that should be executed once the voting on a proposal is completed. Spaces can whitelist an array of execution strategies and then each proposal creator should select one when creating a proposal. All execution strategies should have an external `execute` function with the following interface: 

```
func execute(proposal_outcome : felt, execution_params_len : felt, execution_params : felt*):
end
```

This function is called internally by the space contract when `finalize_proposal` is called. Once the strategy receives the `proposal_outcome` (Accepted, Rejected, Cancelled), along with the `execution_params`, it can perform perform arbitrary logic on the data and execute transactions. For example, it could check `proposal_outcome` to see whether the proposal had passed and if so execute one set of transactions, otherwise executing another set. 

We provide the following strategies inside the sx-core repo:

### [StarkNet Execution](https://github.com/snapshot-labs/sx-core/blob/6420b6ec2e3812822d670adf9857c4b231a1f052/contracts/starknet/lib/voting.cairo#L713)

The space contract builds on top of the OpenZeppelin account standard and is therefore an account contract itself. This means that it has the necessary logic to execute StarkNet transactions already. For this reason, the StarkNet execution strategy is built into the space contract itself and does not reside in a separate contract. This strategy will therefore not have an address like the others so we index it with the value `1`.

One can execute an arbitrary number of transactions with this strategy, the `execution_params` array should encoded as set out [here].(https://github.com/snapshot-labs/sx.js/blob/master/src/utils/encoding/starknet-execution-params.ts) 



Currently this repo provides the Zodiac Execution Strategy. This enables a DAO to perform permissionless execution of L1 Gnosis Safe transactions upon the successful completion of a proposal. The [Zodiac Relayer](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/execution/zodiac\_relayer.cairo%60) contract is the `executor` for this strategy, which will forward the execution to the [L1 Zodiac module](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/ethereum/SnapshotXZodiacModule/SnapshotXL1Executor.sol) address specified in `executions_params[0]`. To use this strategy, DAOs will need a Gnosis Safe with the Snapshot X Zodiac module activated.

We will also be adding a StarkNet transaction execution strategy in the near future.
