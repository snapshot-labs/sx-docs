# Authenticators

Authenticators are the contracts in charge of authenticating users. All authenticators have a function titled `execute` that takes the following arguments:

* `to`: The address of the space contract the user wants to interact with.
* `function selector`: The function selector for the function inside the space contract that the user wants to invoke (vote or propose).
* `calldata`: An array of parameters that are required by the function specified by `function selector`.

Beyond this, each authenticator implements different logic depending on the type of authentication that is being done. This repository provides three useful authenticators:

#### [Ethereum signature authenticator](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/authenticator/ethereum.cairo)&#x20;

Will authenticate a user based on a message signed by Ethereum private keys.

#### [StarkNet signature authenticator](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/authenticator/starknet.cairo)

Will authenticate a user based on a message signed by StarkNet private keys.

#### [Ethereum transaction authenticator](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/starknet/authenticator/l1\_tx.cairo)

Will authenticate a user via getting them to submit a transaction on Ethereum and checking that the sender address is valid. Specifically, the user will call the commit method of the [StarkNet Commit](https://github.com/snapshot-labs/sx-core/blob/develop/contracts/ethereum/L1Interact/StarkNetCommit.sol) L1 contract with a hash of their desired `to`, `function_selector`, and `calldata`. This hash along with the user's Ethereum address will then be sent to the Ethereum Transaction Authenticator by the StarkNet message bridge and will be stored there. The user then submits the hash preimage to the `execute` method of the authenticator and the hash will be computed and checked against the one stored. If the hashes match and the sender address stored corresponds to the address in the `calldata`, then authentication was successful. The core use case for this is to allow smart contract accounts such as multi-sigs to use Snapshot X as they have no way to generate a signature and therefore cannot authenticate via signature verification.

Upon successful authentication of the user, the `execute` method will call the function specified by `function selector` in the space contract `to`, with `calldata` as arguments.

This modularity allows spaces to authenticate users using other authentication methods: For example if you wanted to use Solana keys to authenticate users, you would simply need to write the authenticator contract on StarkNet, whitelist it in your space, and Solana users would be able to vote in your DAO.
