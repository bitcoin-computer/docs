---
order: -40
---

# API

We describe th Api of the class Computer. Objects of the class can create and synchronize to smart objects. It also provides the usual methods of a wallet.

### Constructor

Creates a new Bitcoin Computer wallet.

```js
import { Computer } from 'bitcoin-computer-lib'

const computer = new Computer({
  // BIP39 mnemonic phrase, defaults to a random phrase
  seed, //

  // Target blockchain, defaults to 'LTC'
  chain,

  // Network (one of 'mainnet', 'testnet', or 'regtest'),
  // defaults to 'testnet'
  network,

  // BIP32 path, defaults to "m/44'/0'/0'/0"
  path,

  // Url of a Bitcoin Computer Node, defaults to
  // https://node.bitcoincomputer.io
  url
})
```

## Smart Contract Interface

### new()

Creates new smart objects. The arguments are a class and a list of arguments for the constructor of the class. The arguments can be of basic data type or smart objects.

```js
// A smart contract
class A {
  constructor(n) {
    this.n = n
  }
}

// Create a smart object
const a = await computer.new(A, [1])

// Evaluates to true
expect(a).to.deep.equal({
  n: 1,
  _id: '...',
  _rev: '...',
  _root: '...'
})
```

### sync()

Returns the smart object stored at a given revision.

```js
// Compute smart object from revision
const synced = await computer.sync(a._rev)

// Evaluates to true if "a" is a smart object
expect(synced).to.deep.equal(a)
```

### queryRevs()

Returns an array containing the latest revisions that satisfy certain conditions.

```js
const revs = await computer.queryRevs({
  // Return only revisions owned by a public key
  publicKey: public key,

  // Return only revisions of smart object from a class
  contractName: string,

  // Return only revisions of smart objects from a class by hash
  contractHash: string
})
```

When a key is omitted the condition is ignored. For example, if only ``className`` is set the call will return all revision of that class regardless of the owners.

### idToRev()

Inputs an id and returns the latest revision of the smart object with that id. If no smart object with that id exists an error is thrown.

```js
// Get latest revision
const rev = await computer.idToRev(a._id)

// If a is up to date the next line evaluates to true
expect(rev).to.be(a._rev)
```

## Wallet Interface
### getMenmonic()

Returns a string encoding a BIP93 mnemonic sentence of the ``computer`` object.

```js
const mnemonic = computer.db.wallet.getMnemonic().toString()
```

### getPublicKey()

Returns a string encoded a public key.

```js
const publicKey = computer.db.wallet.getPublicKey().toString()
```

### getAddress()

Returns a string encoded Bitcoin address.

```js
const address = computer.db.wallet.getAddress().toString()
```

### getBalance()

Returns the current balance in satoshi.

```js
const balance = await computer.db.wallet.getBalance()
```

### send()

Sends an amount of satoshi to an address.

```js
const satoshi = 100000
const address = '1FFsHfDBEh57BB1nkeuKAk25H44U7mmMXd'
const balance = await wallet.send(satoshi, address)
```

### broadcast()

**Support will be added in 0.9.0-beta**

Broadcasts a hex encoded Bitcoin transaction to the Bitcoin mining network.

```js
const txHex = '7b1eabe0209b1fe794124575ef807057...'
const txId = await computer.broadcast(txHex)
```


<!--
# Db

The recommended way to create an instance of the Db class is to create an object of the Computer class and to access its property computer.db.

```js
const { computer } = new Computer({ seed, chain, network })
const { db } = computer
```

## db.put()
The ``db.put()`` method inputs an array of JSON objects and stores them in a transaction. Each element of the array is stored in a separate output. The method returns the array of locations of the outputs created.

```js
const data = [{a: 1}, {b: { c: 2 }}]
const locs = await computer.db.put(data)
// locs === ['0322...8dfe:0', '0322...8dfe:1']
```

## db.get()

The ``db.get()`` method returns the JSON objects stored at a given array of locations.

```js
const locs = await computer.db.put(data)
const fromChain = await computer.db.get(locs)
// fromChain === data
```

## db.update()

The ``db.update()`` method has two parameters: a list of locations and a list of JSON objects. It broadcasts a transaction that spends the locations and that has one output for each JSON object.

```js
const locs1 = await computer.db.put([{ n: 1 }])
const locs2 = await computer.db.update(locs1, [{ n: 2 }])
const fromChain = await computer.db.get(locs2)
// fromChain === [{n: 2}]
```

You can use db.get() to inspect the smart contract protocol. Try to call db.get()with the ids or rev of a smart object to see the data on the blockchain. Remember to pass the id inside an array likedb.get([a._id]).


# Wallet

We recommend creating a Wallet instance by creating a Computer instance and accessing the nested wallet property.

```js
const { computer } = new Computer({ seed, chain, network })
const { wallet } = computer.db
```

The wallet built into Bitcoin Computer is compatible with the widely used Bitcore library. This makes it easy to integrate Bitcoin Computer into existing apps.

-->
