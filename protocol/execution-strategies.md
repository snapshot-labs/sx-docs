# Execution Strategies

Execution strategies (also interchangably called executors) are contracts that perform the logic that should be executed once the voting on a proposal is completed. Spaces can whitelist an array of execution strategies and then each proposal creator should select one from the array when creating a proposal. All execution strategies should have an external `execute` function with the following interface: 

```
func execute(proposal_outcome : felt, execution_params_len : felt, execution_params : felt*):
end
```

This function is called internally by the space contract when `finalize_proposal` is called. Once the strategy receives the `proposal_outcome` (Accepted, Rejected, Cancelled), along with the `execution_params`, it can perform arbitrary logic on the data and execute transactions. For example, it could check `proposal_outcome` to see whether the proposal had passed and if so execute one set of transactions, otherwise executing another set. 

We provide the following strategies inside the sx-core repo:

## [Ethereum Execution](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/ExecutionStrategies/ZodiacRelayer.cairo)

This strategy enables Ethereum transactions to be executed following a proposal. It uses the StarkNet message bridge to communicate with an Ethereum destination address. The execution hash is a keccak hash of the transactions within the proposal. We use keccak here so that it can be efficiently computed on Ethereum in order to obtain the transactions that make up the pre-image.

`execution_params` should be set up as follows:
- `execution_params[0]`: Ethereum destination address
- `execution_params[1]`: Proposal outcome
- `execution_params[2]`: Low 128 bits of the keccak execution hash
- `execution_params[3]`: High 128 bits of the keccak execution hash

We provide an Ethereum execution contract: A [Zodiac Module](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/ethereum/ZodiacModule/SnapshotXL1Executor.sol) that enables transactions to be executed from a Gnosis Safe. To use this module, DAOs will need deploy their own proxy and then enable it within their Safe. 

## [StarkNet Execution](https://github.com/snapshot-labs/sx-core/blob/6420b6ec2e3812822d670adf9857c4b231a1f052/contracts/starknet/lib/voting.cairo#L713)

The space contract builds on top of the OpenZeppelin account standard and is therefore an account contract itself. This means that it has the necessary logic to execute StarkNet transactions already. For this reason, the StarkNet execution strategy is built into the space itself and does not reside in a separate contract. This strategy will therefore not have an address like the others so we index it with the value `1`.

One can execute an arbitrary number of transactions with this strategy, the `execution_params` array should encoded as set out [here](https://github.com/snapshot-labs/sx.js/blob/master/src/utils/encoding/starknet-execution-params.ts). 

### And More! 

Feel free to create your own execution strategies! The interface of a strategy can be found [here](https://github.com/snapshot-labs/sx-core/blob/goerli_testing/contracts/starknet/Interfaces/IExecutionStrategy.cairo)



