---
order: -20
---

![](/static/imgs/bitcoin-computer%401x.png)


# Tutorial


## Write a Smart Contract

Smart contracts are Javascript classes that extend from ``Contract``. For example, a smart contract for a chat could be:

```js
import { Contract } from '@bitcoin-computer/lib'

class Chat extends Contract {
  constructor(message) {
    super({ messages: [message] })
  }

  post(message) {
    this.messages.push(message)
  }
}
```

Smart contracts are mostly just Javascript classes. One distiction is that it is not possible to assign to ``this`` in constructors. Instead you can initialize a smart object by passing an argument into ``super`` as shown above.

## Create a Wallet

You can create a smart contract wallet from the Bitcoin Computer library.

```javascript
import { Computer } from '@bitcoin-computer/lib'

const mnemonic = 'replace this seed'
const computer = new Computer({ mnemonic })
```

You can pass in a [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) mnemonic to initialize the wallet. To generate a fresh mnemonic click [here](https://iancoleman.io/bip39/). You can find more wallet configuration options [here](https://docs.bitcoincomputer.io/library/api/#constructor).

## Create a Smart Object

The ``computer.new`` function can be used to create a smart object from a smart contract. For example

```js
const a = await computer.new(Chat, ['hi'])
```

When this call is executed, a transaction is broadcast that contains both the source code of ``Chat`` as well as the expression ``new Chat('hi')``. The object ``a`` that is returned has all the properties defined in the class, and five extra properties ``_id``, ``_rev``, ``_root``, ``_owners`` and ``_amount``.

```js
expect(a).to.deep.equal({
  messages: ['hi'],
  _id: '667c...2357/0',
  _rev: '667c...2357/0',
  _root: '667c...2357/0',
  _owners: ['03...'],
  _amount: 5820
})
```

The additional properties are explained in detail in the [Protocol](/protocol#keyword-properties). For now it is sufficient to know that ``667c...2357`` is the transaction id of the transaction that contains the expression `` `${Chat} new Chat('hi')` ``. The property ``_owners`` is an array of public keys that are allowed to update the object. The property ``_amount`` is the amount of satoshis that are stored in the object.

## Read a Smart Object

The ``computer.sync`` function computes the state of smart object from the metadata on the blockchain. For example, synchronizing to ``667c...2357/0`` it will return an object with the same value as ``a``.

```js
const b = await computer.sync('667c...2357/0')
expect(b).to.deep.equal(a)
```

Reading and writing can be performed to different users. The blockchain allows any two users to obtain consensus over the state of smart objects. This makes it possible to build decentralized applications.

## Update a Smart Object

Smart objects can only be updated through function calls. As function calls are recorded in Bitcoin transactions it is necessary to ``await`` on function calls.

```js
await a.post('yooo')
expect(a).to.deep.equal({
  messages: ['hi', 'yooo'],
  _id: '667c...2357/0',
  _rev: 'de43...818a/0',
  _root: '667c...2357/0',
  _owners: ['03...'],
  _amount: 5820
})
```

Note that ``_rev`` has been update but that ``_id`` and ``_root`` stayed the same. Every time a smart object is updated a new *revision* is created and assigned to the ``_rev`` property. Revisions allow you to reconstruct each historical state of an object.

```js
const oldChat = await computer.sync('667c...2357/0')
expect(oldChat.messages).to.deep.equal(['hi'])

const newChat = await computer.sync('de43...818a/0')
expect(newChat.messages).to.deep.equal(['hi', 'yo'])
```

## Find a Smart Object

The ``computer.query`` function provides several ways of finding the latest revision of a smart object.

```js
const [rev] = await computer.query({ ids: ['667c...2357/0']})
expect(rev).to.equal('de43...818a/0')
```

A basic pattern for many applications is to identify smart objects by their id, look up their latest revision using ``computer.query`` and then to compute their current state using ``computer.sync``. For example, in a chat, we might have the url for the chat containing the id of the chat object. We could then recover the latest state of the chat as follows:

```js
const url = window.location
const parse = (url) => ... // extracts id from url
const id = parse(url)
const [rev] = await computer.query({ ids: [id] })
const obj = await computer.sync(rev)
```

## Data Ownership

Every smart object can have up to three owners. Only an owner can update the object. The owners can be set by assigning string encoded public keys to the ``_owners`` property of a smart object. If the ``_owners`` property is not assigned in a smart contract it defaults to the public key of the computer object that created the smart object.

In our chat example the initial owner is the user that created the chat with ``computer.new``. Thus only this user will be able to post to the chat. We can add a function to update the owners array to invite more guests to chat.

```js
class Chat extends Contract {
  ... // like above

  invite(pubKeyString) {
    this._owners.push(pubKeyString)
  }
}
```

## Privacy

By default, the state of all smart objects is public. However, you can restrict read access to an object by setting a property ``_readers``. If ``_readers`` is assigned to an array of public keys, the meta-data of the current revision is encrypted in a way that only the specified readers can decrypt it.

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

A user can (only) read the state of a smart object if they have read access to the current and all previous versions of the object. It is, therefore, not possible to revoke access to a revision. However, it is possible to remove a user's ability to read the state of a smart object from a point in time forwards.

Both encryption and decryption happen securely in users' browsers. We note that while all smart contract data is encrypted, flows of money are not obfuscated in order not to hinder anti-money laundering efforts.

## Saving Block Space

Not all data needs to be stored on the blockchain. For example, personal data should never be stored on chain, not even encrypted.

When the property ```_url``` of a smart object is set to the URL of a Bitcoin Computer Node, the corresponding metadata is stored on the specified Bitcoin Computer Node. The blockchain only contains a hash of the meta data and a link to where it can be retrieved.

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

Each smart object can store an amount of satoshi. By default a smart object stores a minimal (non-dust) amount. If the ``_amount`` property of a smart object is set to a number, the output storing that smart object will contain that number of satoshis. For example, consider the class ``Payment`` below.

```js
class Payment extends Contract {
  constructor(to: string, amount: number) {
    super()
    this._owners = [to]
    this._amount = amount
  }

  cashOut() {
    this._amount = 0
  }
}
```

If a user ``A`` wants to send 210.000 satoshis to a user ``B``, the user ``A`` can setup the payment as follows:

```js
const computerA = new Computer({ mnemonic: <A's seed phrase> })
const payment = computerA.new(Payment, [<B's public key>, 210000])
```

When the ``payment`` smart object is created, the wallet inside the ``computerA`` object funds the ``210.000`` satoshi that are stored in the ``payment`` object. Once ``B`` becomes aware of the payment, he can withdraw by syncing against the object and calling the ``cashOut`` function.

```js
const computerB = new Computer({ seed: <B's seed phrase> })
const paymentB = await computerB.sync(payment._rev)
await paymentB.cashOut()
```
