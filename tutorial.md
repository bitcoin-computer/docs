---
order: -30
---

# Tutorial

This tutorial explains how smart objects can be created and updated on the Bitcoin blockchain, and how multiple users can synchronize to the same smart object.

## The Computer Object

The first step is to create an object computer from the Bitcoin Computer npm package.
import { Computer } from 'bitcoin-computer'

```javascript
const computer = new Computer({
  seed, // defaults to a random BIP39 mnemonic
  chain, // currently only 'LTC'
  network, // 'mainnet', 'testnet', or 'regtest', defaults to 'testnet'
  path // BIP32 path, defaults to "m/44'/0'/0'/0"
})
```

---

All keys are optional. You can generate a fresh BIP39 seed phrase using a seed phrase generator.

## Creating a Smart Object

```javascript
A smart contract is a Javascript class.
class Counter {
  constructor(n) {
    this.n = n
  }
  inc() {
    this.n += 1
  }
}
```

The ``new()`` method of the computer object creates a smart object from a Javascript class and an array of arguments for the constructor. A Bitcoin transaction is broadcast to record the creation of the object.

```javascript
const counter = await computer.new(Counter, [2])
```

Every smart object has a property _id that is set to the output on the Bitcoin blockchain that records the creation of the smart object. This id remains fixed throughout the lifecycle of the object.

## Updating a Smart Object

A smart object can be updated by calling one of its functions. When a smart object is updated a transaction is broadcast that records the state change. It is therefore necessary to ``await`` on function calls.
!!!
When a smart object is updated a transaction is broadcast that records the state change. It is therefore necessary to awaiton function calls.
!!!

```javascript
await counter.inc()
```

A new output is created that stores the new revision of the smart object. The property counter._rev.is set to the output that stores the latest revision.

## Reading a Smart Object

The ``computer.sync()` method returns a smart object given a revision. This allows one user to create a smart object, send the revision of the smart object to another user, and the second user can synchronize to the same object. For example a user Alice could run the following code

```javascript
import { Computer } from 'bitcoin-computer-lib'

const computerA = new Computer({ seed: 'seed phrase of Alice' })
const counterA = await computerA.new(Counter, [2])

console.log(counterA._rev)
// logs 'ac445f8aa0e6503cf2e...03cf2e/0'
```

Next, Alice sends the revision ``ac445f8aa0e6503cf2e...03cf2e/0`` to her friend Bob. Bob can create an instance of the same smart object on his local machine.

```javascript
import { Computer } from 'bitcoin-computer-lib'

const computerB = new Computer({ seed: 'seed phrase of Bob'})
const revision = 'ac445f8aa0e6503cf2e...03cf2e:0'

const counterB = await computerB.sync(revision)
```

At this point, both Alice and Bob both have a local instance of the same smart object. Additionally, there is an instance of the counter object that is stored in the blockchain. The instance on the blockchain is the global source of truth. If either user has permission to updates the object the other user can recompute the state of the object so that both remain in consensus. Consider, for example, that Alice calls the inc function on her local machine:

```javascript
await counterA.inc()
```

The object on Alice's computer updates the counter to 1. Next, the update is recorded on the blockchain in a transaction. Finally, Bob can sync to the latest revision of the object to compute its latest state. Bob can obtain the latest revision if he knows the owner of the smart object. This concept is explained in the next section.

---

## Data Ownership
Every smart object has a property ``_owners`` that can be set as a list of public key strings. A smart object can only be updated by a user whose public key is in the ``_owners`` array. Specifically, a smart object can only be updated if it was created by a computer instance (either by computer.new or computer.sync ) that contains a mnemonic that corresponds to a public key in the current ``_owners`` property.
The ``_owners`` property can be set in a constructor call and updated in a function call. If no ``_owners`` property is specified, it defaults to a singleton array containing the public key of the current computer object.
The following example shows how the ``_owners`` property can be assigned in construction and function calls.

```javascript
class OwnedCounter {
  constructor(n, pKey) {
    this.n = n
    this._owners = [pKey]
  }
  inc(pKey) {
    this.n += 1
    this._owners = [pKey]
  }
}
```

In the following example Alice can call the inc function but Bob cannot:


```javascript
await counterA.inc()
// works

await counterB.inc()
// throws an error
> Communication Error
> status  400 Bad Request
> message 16: mandatory-script-verify-flag-failed
>   (Operation not valid with the current stack size). Code:-26
```

## Money

A smart object can store an amount of Bitcoin. The amount of Bitcoin, denominated in satoshi, is set through the _amount property. The Bitcoin is owned by the owners of the object, and each owner can send the Bitcoin to another user (like themselves).
As an example consider the class Wallet below.

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

The following example shows how a wallet storing 30000 satoshi is created. The satoshi are taken out of the wallet of the computer object. If a function call reduces the amount of satoshi in an object the remaining satoshi (minus miner fee) are returned to the wallet in the computer object.  The example also shows how the wallet can send Bitcoin.

```javascript
import { Computer } from 'bitcoin-computer-lib'

// create Bitcoin Computer instances
const computerA = new Computer({ seed: 'seed phrase of Alice'})
const computerB = new Computer({ seed: 'seed phrase of Bob'})

const wallet = await computerA.new(Wallet, [30000])
const pKeyB = computerB.db.wallet.getPublicKey().toString()

await wallet.send(20000, pKeyB)
```
