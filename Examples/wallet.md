---
order: 0
---

# Wallet

A wallet allows a user to check their balance and to make a transaction to send cryptocurrency to another user.

In order to build the transaction, we can use a Bitcore-style library. Bitcoin Computer Lib can provide data to the wallet and broadcast the transaction. A [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) mnemonic sentence can be employed to generate a bitcoin computer wallet.

```js
import { Computer } from 'bitcoin-computer-lib'

class Wallet {
  constructor(mnemonic: string) {
    this.computer = new Computer({ mnemonic })
  }

  ...
}
```

Computing the balance is a two-step process. The first step is to find all the unspent transaction outputs (utxos) of the address, which store the cryptocurrency. The second step is to sum up the satoshis in the utxos.

```js
async getBalance(address: Address) {
  const utxos = await this.computer.getUtxos(address)
  return utxos.reduce((prev, curr) => prev + curr.satoshis, 0)
}
```

The easy way to implement the ``send`` function is to use the Bitcoin Computer wallet.

```js
async send(satoshis: number, to: Address) {
  return this.computer.send(satoshis, to)
}
```

## The Hard Way

It is also possible to use the Bitcoin Computer and a Bitcore style library to build a transaction directly. This is interesting because the smart contract for a fungible token is conceptually similar to the code utilized to build a wallet.

The first step is to query for all utxos. We add inputs to the transactions that spend the utxos until we have enough to cover the amount. Then, we add the output to create a new utxo that the receiver can spend. The last thing left to do is to sign and broadcast the transaction.

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

!!!
Check out a working version on [Github](https://github.com/bitcoin-computer/monorepo/tree/main/packages/wallet)
!!!
