---
order: 0
---

!!!
This section explains how a Bitcoin wallet works. The reader interested in smart contracts can skip ahead to the next section.
!!!

# Wallet

A Bitcoin wallet can determine the balance of an address and can build and broadcast a transaction to send cryptocurrency from the current user to another user.

```js
interface Wallet {
  getBalance(address: Address): Promise<number>;
  transfer(to: Address, satoshis: number): Promise<string>
}
```

The Bitcoin Computer library implements the wallet interface.

```js
import { Computer } from 'bitcoin-computer-lib'

...
const computer = new Computer({ mnemonic, url })
const balance = await computer.getBalance(address)
const txId = await computer.send(to, satoshis)
...
```

When the ``send`` function is called the ``computer`` object connects to a Bitcoin Computer Node to broadcast a transaction encoding payment.



The Bitcoin Computer library

also provides a more low level interface that can be used in conjunction with a Bitcore style library.


can also be used in conjunction with a Bitcore-style library to build a transac


 can be used to build the transaction. Bitcoin Computer Lib can be used to provide data to the wallet and to broadcast the transaction. A [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) mnemonic sentence can be used to generate a bitcoin computer wallet.


## The Hard Way

It is also possible to use the Bitcoin Computer and a Bitcore style library to build a transaction directly. This is interesting because the smart contract for a fungible token is conceptually similar to the code used to build a wallet.

The first step is to query for all utxos. We add inputs to the transactions that spend the utxos until we have enough to cover the amount. We then add the output to create a new utxo that the receiver can spend. The last thing left to do it to sign and braodcast the transaction.

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
The "Advanced Examples" Section is are a work in progress. We are using the examples in this section to determine the final syntax and semantics for the Bitcoin Computer.
!!!
