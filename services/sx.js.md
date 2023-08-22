# SX.js

SX.js is the official Typescript SDK to interact with Snapshot X. The SDK includes a client for Mana, the meta transaction relayer.

#### GitHub

[https://github.com/snapshot-labs/sx.js](https://github.com/snapshot-labs/sx.js)

### Cast a vote

{% tabs %}
{% tab title="Ethereum transaction (EVM)" %}
{% code lineNumbers="true" %}
```typescript
import { clients, evmSepolia } from '@snapshot-labs/sx';

const client = new clients.EvmEthereumTx({
  networkConfig: evmSepolia
});

const data = {
  space: '0x012b261effbf548f2b9a495d50b81a8a7c1dd941',
  authenticator: '0xba06e6ccb877c332181a6867c05c8b746a21aed1',
  strategies: [
    {
      address: '0xc1245c5dca7885c73e32294140f1e5d30688c202',
      index: 0
    }
  ],
  proposal: 7,
  choice: 1,
  metadataUri: ''
};

const receipt = await client.vote({
  signer: web3.getSigner(),
  envelope: {
    data
  }
});
console.log('Receipt', receipt);

```
{% endcode %}
{% endtab %}

{% tab title="Ethereum signature (EVM)" %}
TBD
{% endtab %}
{% endtabs %}

{% hint style="info" %}
More details coming soon.
{% endhint %}
