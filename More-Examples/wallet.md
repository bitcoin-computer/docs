---
order: 0
---

# Wallet

A wallet allows a user to check their balance and to build a transaction  to send cryptocurrency to another user.

A Bitcore-style library can be used to build the transaction. Bitcoin Computer Lib can be used to provide data to the wallet and to broadcast the transaction. A [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) mnemonic sentence can be used to generate a bitcoin computer wallet.

```js
import { Computer } from 'bitcoin-computer-lib'

class Wallet {
  constructor(mnemonic: string) {
    this.computer = new Computer({ mnemonic })
  }

  ...
}
```

Computing the balance it a two step process. The first step is to find the utxos of the address. The second step is to sum up the satoshis in the utxos.

```js
async getBalance(address: Address) {
  const utxos = await this.computer.getUtxos(address)
  return utxos.reduce((prev, curr) => prev + curr.satoshis, 0)
}
```

The easy way to implement the send function is to use the Bitcoin Computer wallet.

```js
async send(satoshis: number, to: Address) {
  return this.computer.send(satoshis, to)
}
```

## The Hard Way

It is also possible to use the Bitcoin Computer and a Bitcore style library to build a transaction directly. This is interesting because the smart contract for a fungible token is has conceptual similar to the code used to build a wallet.

The first step is to query for all unspent transaction outputs (utxos). The utxos store the cryptocurrency. We add inputs to the transactions that spend the utxos until we have enough to cover the amount. We then add the output to create a new utxo that the receiver can spend. The last things left to do it to sign and braodcast the transaction.

```js #
async send(satoshis: number, to: Address) {
  const utxos = await this.computer.getUtxos()
  let tx = new Bitcore.Transaction()

  // While we have not added enough inputs to the transaction to cover the amount:
  // add another input and decrement the nummber of satoshis to send
  while(satoshis > 0) {
    const utxo = utxos.splice(0, 1)
    transaction.from(utxo)
    satoshis -= utxo.satoshis
  }

  // Throw error if not enough utxos could be found
  if(satoshis > 0) throw new Error('Not enough funds.')

  // Otherwise add payment output, change output, sign the transaction and broadcast it
  transaction.to(to, amount)
  transaction.change(this.computer.getAddress())
  transaction.sign(this.computer.getPrivateKey())

  // Broadcast
  await this.computer.broadcast(transaction)
}
```

There is one abstraction of the code ab

!!!
The "More Examples" Section is under construction. We are using it to refine our api so some features (``computer.getUtxos`` and ``computer.broadcast``) are not supported yet. See the [Api](../Library/api.md) for a list of currently supported features.
!!!
