---
order: -30
---

# Tutorial

This tutorial explains how to build an encrypted blockchain-based chat. You can download the code from [Npm](https://www.npmjs.com/package/@bitcoin-computer/lib) or [Github](https://github.com/bitcoin-computer/monorepo/tree/main/packages/lib).

## The Computer Object

The first step is to create an object ``computer`` from ``bitcoin-computer-lib``. The ``computer`` object is a wallet that can build and broadcast Bitcoin transactions, which encode smart object creations and updates. You can pass in a [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) seed phrase to initialize the wallet. (To generate a seed phrase, check [this](https://iancoleman.io/bip39/) out.)

```javascript
import { Computer } from 'bitcoin-computer-lib'

const seed = 'replace this seed' // a BIP39 pass phrase
const computer = new Computer({ seed })
```

By default, a ``computer`` object is configured to connect to Litecoin testnet through a publicly available [Bitcoin Computer Node](https://npmjs.com/package/bitcoin-computer-node). See Section API for details on the configuration options.

## Smart Contracts

Every Javascript (ES6) class is a smart contract. For example, a smart contract for a chat could be:

```javascript
class Chat {
  constructor(message) {
    this.messages = [message]
  }

  post(message) {
    this.messages.push(message)
  }
}
```

## Smart Objects

A smart object is a Javascript object that is stored on the Bitcoin blockchain. The ``computer.new()`` method inputs a class and an array of arguments for the constructor of the class, and then returns a smart object from the class.

```javascript
> a = await computer.new(Chat, ['Hi'])
Chat {
  _id: '667c...2357/0',
  _rev: '37e4...18e2/1',
  _root: '0728...ddbe/6',
  messages: ['Hi']
}
```

When a smart object ``a`` is created, a Bitcoin transaction is broadcasted, which records the creation of ``a``. One of the outputs of the transaction is the immutable representation of the smart object on the blockchain. This output is called the *location* of ``a``.

### Object Identity

Each smart object ``a`` has a unique *identity* ``a._id`` that remains fixed throughout the lifecycle of the object. This identity includes the transaction id and the output number of the location of ``a``.

### Object Revision

Each smart object ``a`` has a *revision* ``a._rev`` that changes every time the object is updated. Each version of a smart object has a unique identifier that can be used to recover every previous version of ``a``.

### Object Root

Each smart object ``a`` has a *root* ``a._root``. For a smart object ``a`` created with ``computer.new``, the root ``a._root`` of ``a`` is equal to ``a._id``. However, smart objects can also be created inside a constructor or a function call on another smart object ``b``. In this case, the root of ``a`` is the root of ``b``.

## Updating a Smart Object

Smart objects can be updated through function calls (because it is not possible to assign to a property of a smart object directly). When a function is called, the output representing the smart object before the function call is spent, and a new unspent output (utxo) representing the object after the call is created; this transaction is then broadcasted. Note that it is necessary to ``await`` on function calls as broadcasting a transaction is an asynchronous operation.

```javascript
await a.post('Hi!')
```

 It is important to note that a user can call the function of a smart object ``a`` only if that user can spend the output that stores the current state of ``a``. This is the basis of the security model of the Bitcoin Computer.

## Data Ownership

Every smart object ``a`` contains an array ``a._owners`` of public key strings. This array can be used to restrict write access to the smart object: in order to call a function on ``a``, a user needs a private key matching one of the public keys in the ``a._owners`` array.

It is possible to reassign the ``a._owners`` property to change the ownership of ``a``. If the property is not assigned in function calls, it is unchanged; in constructor calls, it defaults to the public key of the ``computer`` object that creates ``a``.

For example, in our chat, only the user that creates the chat can post initially. However, we can add an invite function to the chat to allow other users to post.

```js
class Chat {
  ... // from above

  invite(pubKeyString) {
    this._owners.push(pubKeyString)
  }
}
```

## Encryption

By default, the state of all smart objects is publicly visible. However, every smart object ``a`` has a property ``a._readers`` that can be used to restrict read access to ``a``. If ``a._readers`` is set, the meta data of the current revision is encrypted so that only the specified readers can decrypt it. If ``a._readers`` is not assigned, it remains unchanged in function calls and defaults to the public key of the ``computer`` object that creates ``a``.

For example, if we want to ensure that only people invited to the chat can read the messages, we can update our example code as follows:

```js
class Chat {
  // ... as above

  invite(pubKey) {
    this._owners.push(pubKey)
    this._readers.push(pubKey)
  }
}
```

A user can (only) read the state of a smart object if they have read access to the current and all previous versions of the object. It is, therefore, not possible to revoke access that has already been granted from smart objects. However, it is possible to remove a user's ability to read the state of a smart object from a point in time forwards.

Both encryption and decryption happen securely in users' browsers. We note that while all smart contract data is encrypted, flows of money are not obfuscated in order not to hinder anti-money laundering efforts.

## Finding Smart Objects

The process of reading the current state of a smart objects ``a`` consists of two steps: finding the latest revision of ``a`` and synchronizing to the revision (synchronizing will be described in the next Section).

### Querying by Ownership

The ``computer.queryRevs()`` method returns an array of all revisions that satisfy certain conditions as specified in the parameter. For example, one can obtain all revisions owned by a public key or all revisions of a specific smart contract.

```js
const revs1 = await computer.queryRevs({ pubKey })
const revs2 = await computer.queryRevs({ contractHash })
const revs3 = await computer.queryRevs({ pubKey, contractHash })
```

### Querying by Identity

It is often convenient to refer to a smart object by its identity. In order to synchronize to the latest version of the object, its latest revision is needed. The ``computer.idToRev()`` function returns the latest revision of a given id.

```js
const [rev] = await computer.idToRev([id])
```
Multiple ids can be passed in, and their revisions will be returned in that order.

## Synchronizing to a Smart Object

The ``computer.sync()`` method returns a smart object given a revision.

```js
const b = await computer.sync(rev)
```

In the example of our decentralized chat, a user can first synchronize to the chat to read the messages. If the user is an owner, then the user can post a message.

```js
const [rev] = await computer.query({ className: 'Chat' })
const chat = await computer.sync(rev)
await chat.post('Hello')
```

## Off-Chain Storage

It is sometimes not the best choice to store data on the blockchain. For example, if there's a large amount of data, it might be too expensive. Other data, like personal data, should never be stored on chain, not even encrypted, in order to comply with end user privacy regulations, such as CCPA and GDPR.

Each smart object ``a`` has a property ``a._url`` that can be set to the url of a Bitcoin Computer Node. When this property is set, the meta data that encodes an update is not stored on the blockchain but on the Bitcoin Computer Node at the given url. The blockchain contains only the hash of the meta data and a link to where the data can be obtained.

For example, if we want to allow users to send images that are too large to be stored on chain to the chat, we can make use of the off-chain solution:

```js
class Chat {
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

The meta data for that specific call will be stored on the Bitcoin Computer Node at ``https://my.node.example.com``. The only data that is stored on the blockchain is the url and the hash of the data. The cost of calling ``postPic`` is thus independent of the size of ``picBuf``.

## Sending and Storing Cryptocurrency

Each smart object can store an amount of cryptocurrency. By default a smart object stores a minimal (non-dust) amount. The smart contract developper can set an amount by assigning an integer to a property ``_amount`` of a smart object.

For example, consider the class ``Payment`` below.

```js
class Payment {
  constructor(to: string, amount: number) {
    this._owners = [to]
    this._amount = amount
  }

  cashOut() {
    this._amount = 10000
  }
}
```

If a user ``A`` wants to send 200000 satoshis to a user ``B``, the user ``A`` can setup the payment as follows:

```js
const computerA = new Computer({ seed: <A's seed phrase> })
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

At the end of the process 210000-10000= 200000 many additional satoshis will be in the wallet with seed ``<B's seed phrase>``.


<!--As an example, consider the class ```Wallet``` below.

```javascript
class Wallet {
  constructor(amount) {
    this._amount = amount
  }

  send(amount, to) {
    this._amount = amount
    this._owners = [to]
  }
}
```

The following example shows how a wallet storing 30000 satoshis is created. The satoshis are taken out of the wallet of the ```computer``` object. If a function call reduces the amount of satoshis in an object, the remaining satoshis (minus miner fee) are then returned to the wallet in the ```computer``` object.  This example also shows how the wallet can send Bitcoins.

```javascript
import { Computer } from 'bitcoin-computer-lib'

// create Bitcoin Computer instances
const computerA = new Computer({ seed: 'seed phrase of Alice'})
const computerB = new Computer({ seed: 'seed phrase of Bob'})

const wallet = await computerA.new(Wallet, [30000])
const pKeyB = computerB.db.wallet.getPublicKey().toString()

await wallet.send(20000, pKeyB)
```
-->
