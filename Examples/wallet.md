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



It is also possible to use the Bitcoin Computer with a Bitcore style library.

```js
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
