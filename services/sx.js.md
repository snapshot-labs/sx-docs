# SX.js

SX.js is the official Typescript SDK to interact with Snapshot X. The SDK includes a client for Mana, the meta transaction relayer.

#### GitHub

[https://github.com/snapshot-labs/sx-monorepo/tree/master/packages/sx.js](https://github.com/snapshot-labs/sx-monorepo/tree/master/packages/sx.js)

## Quick start

### Installation

Install the latest version of the beta release:

{% tabs %}
{% tab title="NPM" %}
```bash
npm install @snapshot-labs/sx@beta
```
{% endtab %}

{% tab title="Yarn" %}
```bash
yarn add @snapshot-labs/sx@beta
```
{% endtab %}

{% tab title="Bun" %}
```bash
bun add @snapshot-labs/sx@beta
```
{% endtab %}
{% endtabs %}

### Configuration

#### Clients

Everything happens thanks to the `clients` objects. Depending on the specific Space or Proposal setup, you may need to use a Transaction or Signature Client. This quick guide demonstrates how to easily set all of them up, both for Ethereum and StarkNet:

{% tabs %}
{% tab title="EVM clients" %}
```typescript
import { clients, evmSepolia } from '@snapshot-labs/sx';

const clientConfig = { networkConfig: evmSepolia }

const client = new clients.EthereumTx(clientConfig);
const ethSigClient = new clients.EthereumSig(clientConfig);
```
{% endtab %}

{% tab title="Starknet clients" %}
```typescript
import { clients } from '@snapshot-labs/sx';
import { Provider, constants } from 'starknet';

const ethUrl = 'https://rpc.snapshot.org/1';
const manaUrl = 'https://mana.pizza';

const starkProvider = new Provider({ sequencer: { network: constants.NetworkName.SN_GOERLI } });

const clientConfig = {
  starkProvider,
  manaUrl,
  ethUrl
};

const client = new clients.StarkNetTx(clientConfig);
const starkSigClient = new clients.StarkNetSig(clientConfig);
```
{% endtab %}
{% endtabs %}

To learn more about StarkNet's `Provider` object, have a look [here](https://www.starknetjs.com/docs/guides/connect\_network).

## Usage

{% hint style="info" %}
Make sure to use the right client for your use case. The examples below use Sepolia network and its config is imported from the `sx.js` package.
{% endhint %}

### Cast a vote

{% tabs %}
{% tab title="Ethereum transaction (EVM)" %}
<pre class="language-typescript" data-line-numbers><code class="lang-typescript">import { clients, evmSepolia } from '@snapshot-labs/sx';
import type { Web3Provider } from '@ethersproject/providers';

const clientConfig = { networkConfig: evmSepolia }
const client = new clients.EthereumTx(clientConfig);

const web3 = new Web3Provider(window.ethereum);

<strong>const data = {
</strong>  space: '0x012b261effbf548f2b9a495d50b81a8a7c1dd941',
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
</code></pre>
{% endtab %}

{% tab title="Ethereum signature (EVM)" %}
```typescript
import { clients, evmSepolia } from '@snapshot-labs/sx';
import type { Web3Provider } from '@ethersproject/providers';

const clientConfig = { networkConfig: evmSepolia }
const ethSigClient = new clients.EthereumSig(clientConfig);

const web3 = new Web3Provider(window.ethereum);

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

const receipt = await ethSigClient.vote({
  signer: web3.getSigner(),
  data
});

console.log('Receipt', receipt);
```
{% endtab %}

{% tab title="Starknet transaction" %}
```typescript
import { clients } from '@snapshot-labs/sx';
import { Provider, constants } from 'starknet';

const web3 = window.starknet.provider;

const ethUrl = 'https://rpc.snapshot.org/1';
const manaUrl = 'https://mana.pizza';

const starkProvider = new Provider({ sequencer: { network: constants.NetworkName.SN_GOERLI } });

const clientConfig = {
  starkProvider,
  manaUrl,
  ethUrl
};

const client = new clients.StarkNetTx(clientConfig);

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

const receipt = await client.vote(web3.provider.account, {
    data
});

console.log('Receipt', receipt);
```
{% endtab %}

{% tab title="Starknet signature" %}
```typescript
import { clients } from '@snapshot-labs/sx';
import { Provider, constants } from 'starknet';

const web3 = window.starknet.provider;

const ethUrl = 'https://rpc.snapshot.org/1';
const manaUrl = 'https://mana.pizza';

const starkProvider = new Provider({ sequencer: { network: constants.NetworkName.SN_GOERLI } });

const clientConfig = {
  starkProvider,
  manaUrl,
  ethUrl
};

const client = new clients.StarkNetSig(clientConfig);

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

const envelope = await client.vote({
    signer: web3.provider.account,
    data
});

const receipt = await client.send(envelope);

console.log('Receipt', receipt);
```
{% endtab %}
{% endtabs %}

### Create a proposal

{% tabs %}
{% tab title="Ethereum transaction (EVM)" %}
```typescript
import { clients, evmSepolia } from '@snapshot-labs/sx';
import type { Web3Provider } from '@ethersproject/providers';

const clientConfig = { networkConfig: evmSepolia }
const client = new clients.EthereumTx(clientConfig);

const web3 = new Web3Provider(window.ethereum);

const data = {
  space: '0x012b261effbf548f2b9a495d50b81a8a7c1dd941',
  authenticator: '0xba06e6ccb877c332181a6867c05c8b746a21aed1',
  strategies: [
    {
      address: '0xc1245c5dca7885c73e32294140f1e5d30688c202',
      index: 0
    }
  ],
  executionStrategy: {
    addr: '0x0000000000000000000000000000000000000000',
    params: '0x'
  },
  metadataUri: ''
};

const receipt = await client.propose({
  signer: web3.getSigner(),
  envelope: {
    data
  }
});

console.log('Receipt', receipt);
```
{% endtab %}

{% tab title="Ethereum signature (EVM)" %}
```typescript
import { clients, evmSepolia } from '@snapshot-labs/sx';
import type { Web3Provider } from '@ethersproject/providers';

const clientConfig = { networkConfig: evmSepolia }
const client = new clients.EthereumSig(clientConfig);

const web3 = new Web3Provider(window.ethereum);

const data = {
  space: '0x012b261effbf548f2b9a495d50b81a8a7c1dd941',
  authenticator: '0xba06e6ccb877c332181a6867c05c8b746a21aed1',
  strategies: [
    {
      address: '0xc1245c5dca7885c73e32294140f1e5d30688c202',
      index: 0
    }
  ],
  executionStrategy: {
    addr: '0x0000000000000000000000000000000000000000',
    params: '0x'
  },
  metadataUri: ''
};

const receipt = await ethSigClient.propose({
  signer: web3.getSigner(),
    data
});

console.log('Receipt', receipt);
```
{% endtab %}

{% tab title="Starknet transaction" %}
```typescript
import { clients } from '@snapshot-labs/sx';
import { Provider, constants } from 'starknet';

const web3 = window.starknet.provider;

const ethUrl = 'https://rpc.snapshot.org/1';
const manaUrl = 'https://mana.pizza';

const starkProvider = new Provider({ sequencer: { network: constants.NetworkName.SN_GOERLI } });

const clientConfig = {
  starkProvider,
  manaUrl,
  ethUrl
};

const client = new clients.StarkNetTx(clientConfig);

const data = {
  space: '0x012b261effbf548f2b9a495d50b81a8a7c1dd941',
  authenticator: '0xba06e6ccb877c332181a6867c05c8b746a21aed1',
  strategies: [
    {
      address: '0xc1245c5dca7885c73e32294140f1e5d30688c202',
      index: 0
    }
  ],
  executionStrategy: {
    addr: '0x0000000000000000000000000000000000000000',
    params: '0x'
  },
  metadataUri: ''
};

const receipt = await client.propose(web3.provider.account, {
    data
});

console.log('Receipt', receipt);
```
{% endtab %}

{% tab title="Starknet signature" %}
```typescript
import { clients } from '@snapshot-labs/sx';
import { Provider, constants } from 'starknet';

const web3 = window.starknet.provider;

const ethUrl = 'https://rpc.snapshot.org/1';
const manaUrl = 'https://mana.pizza';

const starkProvider = new Provider({ sequencer: { network: constants.NetworkName.SN_GOERLI } });

const clientConfig = {
  starkProvider,
  manaUrl,
  ethUrl
};

const client = new clients.StarkNetSig(clientConfig);

const data = {
  space: '0x012b261effbf548f2b9a495d50b81a8a7c1dd941',
  authenticator: '0xba06e6ccb877c332181a6867c05c8b746a21aed1',
  strategies: [
    {
      address: '0xc1245c5dca7885c73e32294140f1e5d30688c202',
      index: 0
    }
  ],
  executionStrategy: {
    addr: '0x0000000000000000000000000000000000000000',
    params: '0x'
  },
  metadataUri: ''
};

const envelope = await client.propose({
    signer: web3.provider.account,
    data
});

const receipt = await client.send(envelope);

console.log('Receipt', receipt);
```
{% endtab %}
{% endtabs %}

### Update a proposal

{% tabs %}
{% tab title="Ethereum transaction (EVM)" %}
```typescript
import { clients, evmSepolia } from '@snapshot-labs/sx';
import type { Web3Provider } from '@ethersproject/providers';

const clientConfig = { networkConfig: evmSepolia }
const client = new clients.EthereumTx(clientConfig);

const web3 = new Web3Provider(window.ethereum);

const data = {
  space: '0x012b261effbf548f2b9a495d50b81a8a7c1dd941',
  proposal: 1, // proposalId
  authenticator: '0xba06e6ccb877c332181a6867c05c8b746a21aed1',
  executionStrategy: {
    addr: '0x0000000000000000000000000000000000000000',
    params: '0x'
  },
  metadataUri: ''
};

const receipt = await client.updateProposal({
  signer: web3.getSigner(),
  envelope: {
    data
  }
});

console.log('Receipt', receipt);
```
{% endtab %}

{% tab title="Ethereum signature (EVM)" %}
```typescript
import { clients, evmSepolia } from '@snapshot-labs/sx';
import type { Web3Provider } from '@ethersproject/providers';

const clientConfig = { networkConfig: evmSepolia }
const client = new clients.EthereumSig(clientConfig);

const web3 = new Web3Provider(window.ethereum);

const data = {
  space: '0x012b261effbf548f2b9a495d50b81a8a7c1dd941',
  proposal: 1, // proposalId
  authenticator: '0xba06e6ccb877c332181a6867c05c8b746a21aed1',
  executionStrategy: {
    addr: '0x0000000000000000000000000000000000000000',
    params: '0x'
  },
  metadataUri: ''
};

const receipt = await ethSigClient.updateProposal({
  signer: web3.getSigner(),
  envelope: {
    data
  }
});

console.log('Receipt', receipt);
```
{% endtab %}

{% tab title="Starknet transaction" %}
```typescript
import { clients } from '@snapshot-labs/sx';
import { Provider, constants } from 'starknet';

const web3 = window.starknet.provider;

const ethUrl = 'https://rpc.snapshot.org/1';
const manaUrl = 'https://mana.pizza';

const starkProvider = new Provider({ sequencer: { network: constants.NetworkName.SN_GOERLI } });

const clientConfig = {
  starkProvider,
  manaUrl,
  ethUrl
};

const client = new clients.StarkNetTx(clientConfig);

const data = {
  space: '0x012b261effbf548f2b9a495d50b81a8a7c1dd941',
  proposal: 1, // proposalId
  authenticator: '0xba06e6ccb877c332181a6867c05c8b746a21aed1',
  executionStrategy: {
    addr: '0x0000000000000000000000000000000000000000',
    params: '0x'
  },
  metadataUri: ''
};

const receipt = await client.updateProposal(web3.provider.account, {
    data
});

console.log('Receipt', receipt);
```
{% endtab %}

{% tab title="Starknet signature" %}
```typescript
import { clients } from '@snapshot-labs/sx';
import { Provider, constants } from 'starknet';

const web3 = window.starknet.provider;

const ethUrl = 'https://rpc.snapshot.org/1';
const manaUrl = 'https://mana.pizza';

const starkProvider = new Provider({ sequencer: { network: constants.NetworkName.SN_GOERLI } });

const clientConfig = {
  starkProvider,
  manaUrl,
  ethUrl
};

const client = new clients.StarkNetSig(clientConfig);

const data = {
  space: '0x012b261effbf548f2b9a495d50b81a8a7c1dd941',
  proposal: 1, // proposalId
  authenticator: '0xba06e6ccb877c332181a6867c05c8b746a21aed1',
  executionStrategy: {
    addr: '0x0000000000000000000000000000000000000000',
    params: '0x'
  },
  metadataUri: ''
};

const envelope = await client.updateProposal({
    signer: web3.provider.account,
    data
});

const receipt = await client.send(envelope);

console.log('Receipt', receipt);
```
{% endtab %}
{% endtabs %}

### Get voting power

Address' voting power depends on the strategy used. Below you can see an example of getting voting power for whitelist Voting strategy on Starknet.

{% tabs %}
{% tab title="Starknet Whitelist" %}
```typescript
import {
  starknetGoerli1,
  getStarknetStrategy,
} from '@snapshot-labs/sx';

const clientConfig = {
  starkProvider,
  ethUrl,
  networkConfig: starknetGoerli1
};

const address = '0xe3ca14dcb7862116bbbe4331a9927c6693b141aa8936bb76e2bdfa4b551a52';
const voterAddress = '0x556B14CbdA79A36dC33FcD461a04A5BCb5dC2A70';
const strategyMetadata = {
    "tree": [
        {
            "type": 1,
            "address": "0x556B14CbdA79A36dC33FcD461a04A5BCb5dC2A70",
            "votingPower": "10"
        }
    ]
}
const timestamp = 1703090187
const strategyParams = [
    "0xe3ca14dcb7862116bbbe4331a9927c6693b141aa8936bb76e2bdfa4b551a52"
]

const strategy = getStarknetStrategy(address, clientConfig.networkConfig);

const vp = await strategy.getVotingPower(
  address,
  voterAddress,
  strategyMetadata,
  timestamp,
  strategyParams,
  clientConfig
);
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
More details coming soon.
{% endhint %}
