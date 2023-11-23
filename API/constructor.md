---
order: 100
---

# constructor

An instance of the ``Computer`` class is a smart contract enabled wallet. It has the usual features of a wallet like sending payments or checking the balance of an address. In addition it can create smart objects by encoding these in Bitcoin transactions and broadcasting them to the blockchain.

### Syntax
```js
const computer = new Computer(config)
```

### Types

````js
new (config: {
  chain?: 'LTC' | 'BTC' | 'DOGE',
  network?: 'mainnet' | 'testnet' | 'regtest',
  mnemonic?: string,
  path?: string,
  passphrase?: string
  url?: string,
  satPerByte: number
  dustRelayFee: number
} = {}) => Computer
````

### Parameters

#### config
An object with the basic configuration parameters to create the Computer instance.
<div align="center" style="font-size: 14px;">
  <table>
    <tr>
      <th>config</th>
      <th>default value</th>
      <th>description</th>
    </tr>
    <tr>
      <td>chain</td>
      <td>"LTC"</td>
      <td>Target blockchain. Values in 'LTC' or 'BTC'</td>
    </tr>
    <tr>
      <td>network</td>
      <td>"testnet"</td>
      <td>Target network. Values in 'testnet', 'regtest' or 'mainnet'</td>
    </tr>
    <tr>
      <td>mnemonic</td>
      <td>Defaults to a random phrase</td>
      <td>BIP39 mnemonic phrase</td>
    </tr>
    <tr>
      <td>path</td>
      <td>"m/44'/0'/0'"</td>
      <td>BIP32 path</td>
    </tr>
    <tr>
      <td>passphrase</td>
      <td>""</td>
      <td>BIP32 passphrase</td>
    </tr>
    <tr>
      <td>url</td>
      <td>'https://node.bitcoincomputer.io'</td>
      <td>Url of a Bitcoin Computer Node</td>
    </tr>
    <tr>
      <td>satPerByte</td>
      <td>2</td>
      <td>Fee in satoshi per byte</td>
    </tr>
    <tr>
      <td>dustRelayFee</td>
      <td>Defaults are set to the standard on each blockchain. <br> LTC: 30.000, BTC: 3.000</td>
      <td>Dust relay fee</td>
    </tr>
  </table>
</div>

### Return Value

An instance of the Computer class

### Examples
```ts
import { Computer } from '@bitcoin-computer/lib'

// Bitcoin Computer wallet on LTC testnet
const computer1 = new Computer()

// Bitcoin Computer wallet on BTC testnet
const computer2 = new Computer({
  chain: 'BTC'
})

// Bitcoin Computer wallet with custom parameters
const computer3 = new Computer({
  chain: 'LTC'
  network: 'regtest',
  mnemonic: 'replace this seed'
  path: "m/44'/0'/0'/0",
  url: 'https://my-ltc-node.com',
  satPerByte: 1
})
```
