# Authenticators

Authenticators are the contracts in charge of authenticating users to create proposals and cast votes. All proposal creation and vote transactions must be sent to the relevant DAO's space contract via an authenticator using the external `authenticate` method which exists in every authenticator. DAOs are free to write their own custom authenticators that suit their own needs however we provide a number of standard ones that should cover some popular usecases and can be extended if needed. 


The voter or proposer address resides at `calldata[0]` so in general the role of the authenticator is to check that the owner of the `calldata[0]` address authorized the transaction specified.

### [Ethereum signature authenticator](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/Authenticators/EthSig.cairo)&#x20;

Will authenticate a user based on a message signed by an Ethereum private key. Users create an EIP712 signature for a transaction which is checked for validity here.  

```
@external
func authenticate(
    r : Uint256,
    s : Uint256,
    v : felt,
    salt : Uint256,
    target : felt,
    function_selector : felt,
    calldata_len : felt,
    calldata : felt*,
) -> ():
```

### [Ethereum transaction authenticator](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/Authenticators/EthTx.cairo)

Will authenticate a user via getting them to submit a transaction on Ethereum and checking that the sender address is valid. Specifically, the user will call the `commit` method of the [StarkNet Commit](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/ethereum/L1Interact/StarkNetCommit.sol) L1 contract with a hash of their desired `target`, `function_selector`, and `calldata`. 

```
function commit(uint256 _hash) external;
```

This hash along with the user's Ethereum address will then be sent to the the L1 message handler method of authenticator by the StarkNet message bridge, which will store the data in state. 

```
@l1_handler
func commit(from_address : felt, sender : Address, hash : felt) -> ():
```

The user then submits the hash preimage to the `authenticate` method of the authenticator and the hash will be computed and checked against the one stored. If the hashes match and the sender address stored corresponds to the address in the `calldata`, then authentication was successful.

```
@external
func authenticate(target : felt, function_selector : felt, calldata_len : felt, calldata : felt*) -> ():
```

The core use case for this authenticator is to allow smart contract accounts such as multi-sigs to use Snapshot X as they have no way to generate a signature and therefore cannot authenticate via signature verification.


### [StarkNet signature authenticator](https://github.com/snapshot-labs/sx-core/blob/session_key_auth/contracts/starknet/Authenticators/StarkSig.cairo)

Will authenticate a user based on a message signed by a StarkNet private key. EIP191 style signatures are used here. 

```
@external
func authenticate(
    r : felt,
    s : felt,
    salt : felt,
    target : felt,
    function_selector : felt,
    calldata_len : felt,
    calldata : felt*,
) -> ():
```

### [StarkNet Transaction authenticator](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/Authenticators/StarkTx.cairo)

Will authenticate a user by checking the caller address. 

```
@external
func authenticate(target : felt, function_selector : felt, calldata_len : felt, calldata : felt*) -> ():
```

### And More!

Our modular approach here allows spaces to authenticate users using other authentication methods without any changes to the space contract. For example if you wanted to use Solana keys to authenticate users, you would simply need to write the authenticator contract on StarkNet, whitelist it in your space, and Solana users would be able to vote in your DAO.
