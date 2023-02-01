---
order: -20
---

![](/static/imgs/bitcoin-computer%401x.png)


# Tutorial


## Write a Smart Contract

Every Javascript (ES6) class that extends from ``Contract`` is a smart contract. For example, a smart contract for a chat could be:

```js
import { Contract } from '@bitcoin-computer/lib'

class Chat extends Contract {
  constructor(message) {
    super()
    this.messages = [message]
  }

  post(message) {
    this.messages.push(message)
  }
}
```

## Create a Wallet

Next, create a smart contract wallet from the Bitcoin Computer library.

```javascript
import { Computer } from '@bitcoin-computer/lib'

const mnemonic = 'replace this seed'
const computer = new Computer({ mnemonic })
```

You can pass in a [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) mnemonic to initialize the wallet. You can generate a mnemonic [here](https://iancoleman.io/bip39/). See [here](https://docs.bitcoincomputer.io/library/api/#constructor) for more configuration options.

## Create a Smart Object

The ``computer.new`` function can be used to create an object from a smart contract and to store it on the blockchain. For example

```js
const a = await computer.new(Chat, ['hi'])
```

When ``computer.new`` is called with these parameters a transaction is broadcast that contains the expression `` `${Chat} new Chat('hi')` `` to record that the constructor was called. The object ``a`` that is returned to the user has all the properties defined in the class, and three extra properties ``_id``, ``_rev``, ``_root``.

```js
expect(a).to.deep.equal({
  messages: ['hi'],
  _id: '667c...2357/0',
  _rev: '667c...2357/0',
  _root: '667c...2357/0'
})
```

All three properties are set to the same value: ``667c...2357/0``. We will explain these properties in detail later, for now it is sufficient to know that ``667c...2357`` is the transaction id of the transaction that contains the expression `` `${Chat} new Chat('hi')` ``.

## Read a Smart Object

The ``computer.sync`` function can be used to parse the expressions of the blockchain back into Javascript object. For example, if we call ``computer.sync`` with the string ``667c...2357/0`` it will recover the original object:

```js
const b = await computer.sync('667c...2357/0')
expect(b).to.deep.equal(a)
```

We note that reading and writing can be performed to different users. The blockchain allows them to obtain consensus over the state of smart objects. This makes it possible to build decentralized applications.

## Update a Smart Object

A smart object can be updated through function calls. However as function calls are recorded in Bitcoin transactions as well it is necessary to await on function calls:

```js
await a.post('yo')
expect(a).to.deep.equal({
  messages: ['hi', 'yo'],
  _id: '667c...2357/0',
  _rev: 'de43...818a/0',
  _root: '667c...2357/0'
})
```

We can see that the messages array was updated as expected.

We can also see that ``_rev`` has been update but that ``_id`` and ``_root`` stayed the same. Every time a smart object is updated a new *revision* is created and assigned to the ``_rev`` property. We will explain the ``_id`` and the ``_root`` property in detail later, for now we note that they remain fixed throughout the lifecycle of an object.

Revisions allow you to query each historical state of an object:

```js
const oldChat = await computer.sync('667c...2357/0')
expect(oldChat.messages).to.deep.equal(['hi'])

const newChat = await computer.sync('de43...818a/0')
expect(newChat.messages).to.deep.equal(['hi', 'yo'])
```

## Find Smart Objects

The ``computer.query`` function provides a simple way of finding the latest revisions of smart objects.

```js
const [rev] = await computer.query({ ids: ['667c...2357/0']})
expect(rev).to.equal('de43...818a/0')
```

A basic pattern is to identify smart objects by their id, look up their latest revision using ``computer.query`` and then to compute their latest state using ``computer.sync``. For example, in a chat, we might have the url for the chat contain the id of the chat object. We could then recover the latest state of the chat as follows:

```js
const url = window.location
const parse = (url) => ... // extracts id from url
const id = parse(url)
const rev = await computer.query({ ids: [id] })
const obj = await computer.sync(rev)
```

## Data Ownership

Every smart object has one or more owners and only the owners can update the object. The owners can be set by assigning string encoded public keys to the ``_owners`` property. If the ``_owners`` property is not assigned in a smart contract it defaults to the public key of the computer object that created the smart object.

For example in our chat example the only owner will be the user that created the chat with ``computer.new``. Thus only that user will be able to post to the chat which is a little boring. So we can add a function to update the owners array to invite more guests to chat.

```js
class Chat extends Contract {
  ... // from above

  invite(pubKeyString) {
    this._owners.push(pubKeyString)
  }
}
```

## Privacy

By default, the state of all smart objects is publicly visible. However, smart objects have a property ``_readers`` that can be used to restrict read access to the object. If ``_readers`` is assigned to an array of public keys, the meta-data of the current revision is encrypted in a way that only the specified readers can decrypt it. If ``_readers`` is not set, it defaults to the public key of the "computer" object that created the object.

For example, if we want to ensure that only people invited to the chat can read the messages, we can update our example code as follows:

```js
class Chat extends Contract {
  // ... as above

  invite(pubKey) {
    this._owners.push(pubKey)
    this._readers.push(pubKey)
  }
}
```

A user can (only) read the state of a smart object if they have read access to the current and all previous versions of the object. It is, therefore, not possible to revoke access that has already been granted from smart objects. However, it is possible to remove a user's ability to read the state of a smart object from a point in time forwards.

Both encryption and decryption happen securely in users' browsers. We note that while all smart contract data is encrypted, flows of money are not obfuscated in order not to hinder anti-money laundering efforts.

## Saving Block Space

It is sometimes not the best choice to store data on the blockchain. For example, if there's a large amount of data, it might be too expensive. Other data, like personal data, should never be stored on chain, not even encrypted, in order to comply with privacy regulations, such as CCPA and GDPR.

The property ```_url``` of each smart object can be used to specify the URL of a Bitcoin Computer Node. When this property is set, the meta data that encodes an update is not recorded on the blockchain, but instead is stored on the specified Bitcoin Computer Node. The blockchain only contains a hash of the meta data and a link to where it can be retrieved.

For example, if we want to allow users to send images that are too large to be stored on chain to the chat, we can make use of the off-chain solution:

```js
class Chat extends Contract {
  // ... as above

  post(message) {
    this._url = null
    this.messages.push(message)
  }

  postPic(picBuf) {
    this._url = 'https://my.bc.node.example.com'
    this.messages.push(picBuf)
  }
}
```

## Cryptocurrency

Each smart object can store an amount of cryptocurrency. By default a smart object stores a minimal (non-dust) amount. The smart contract developer can set an amount by assigning an integer to a property ``_amount`` of a smart object.

For example, consider the class ``Payment`` below.

```js
class Payment extends Contract {
  constructor(to: string, amount: number) {
    super()
    this._owners = [to]
    this._amount = amount
  }

  cashOut() {
    this._amount = 10000
  }
}
```

If a user ``A`` wants to send 210000 satoshis to a user ``B``, the user ``A`` can setup the payment as follows:

```js
const computerA = new Computer({ mnemonic: <A's seed phrase> })
const payment = computerA.new(Payment, [<B's public key>, 210000])
```

When the ``payment`` smart object is created, the wallet inside the ``computerA`` object funds the ``210000`` satoshi that are stored in the ``payment`` object.

Generally, constructor and function calls with no parameters work in the same way: The ``computer`` object that either created or synced against the smart object has to cover the satoshis in a smart object without parameters.

In the case of a constructor or function call with parameters we first check if the parameters contain enough funds to cover the amount in the parameters after the call and the amount(s) in the return value. If so, no additional fees from the "current" wallet is needed. If the amount in the parameters exceeds the amount specified in the smart object the funds are sent back to the "current" wallet.

In the next step, ``A`` can send send ``payment._rev`` to user ``B``. To claim the funds user ``B`` can execute:

```js
const computerB = new Computer({ seed: <B's seed phrase> })
const paymentB = await computerB.sync(payment._rev)
await paymentB.cashOut()
```

At the end of the process 210000-10000=200000 many additional satoshis will be in the wallet with seed ``<B's seed phrase>``.
