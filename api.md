---
order: -40
---

# Bitcoin Computer Library Api

This Api describes the class Computer that can create and synchronize to smart objects, the class DB that can read, write, and update data on the blockchain, and the class Wallet, a Bitcoin wallet.

# Computer

## Constructor

```javascript
import { Computer } from 'bitcoin-computer-lib'

const computer = new Computer({
  // BIP39 mnemonic phrase, defaults to a random phrase
  seed,

  // Defaults to 'LTC', support for other chains soon
  chain,

  // 'mainnet', 'testnet', or 'regtest', defaults to 'testnet'
  network,

  // BIP32 path, defaults to "m/44'/0'/0'/0"
  path,

  // Url of a Bitcoin Computer Node, defaults to https://node.bitcoincomputer.io
  url
})
```

## computer.new()

The ``computer.new()`` method creates new smart objects from a class and a list of arguments for the constructor of the class. The arguments can be basic data types or smart objects.

```javascript
class A {
  constructor(n) {
    this.n = n
  }
}

const a = await computer.new(A, [1])
```



## computer.sync()

The ``computer.sync()`` method returns the smart object stored at a given revision. For every smart object ``a`` the following invariant will be true.

```javascript
const b = await computer.sync(a._rev)
expect(b).to.deep.equal(a)  // true
```

## computer.queryRevs()

The ``computer.queryRevs()`` method returns an array containing the latest revisions owned by a given public key. The public keys are string encoded. If no parameter is passed to getRevs() the public key of the computer object is used.

```javascript
const revs = await computer.queryRevs({
  // Return only revisions owned by a public key
  publicKey: public key,

  // Return only revisions of smart object from a class
  contractName: string,

  // Return only revisions of smart objects from a class by hash
  contractHash: string
})
```

When a key is omitted the condition is ignored. For example, if only the class is set the call will return all revision of that class regardless of the owners.

## computer.getLatestRevs()

The ``computer.getLatestRevs()`` method inputs an id and returns the latest revision of the smart object with that id. If no smart object with that id exists an error is thrown.

```javascript
const rev = await computer.getLatestRevs(a._id)
```

After running the above code ``rev`` will be equal to ``a._rev`` if a1 is a smart object.

# Db

The recommended way to create an instance of the Db class is to create an object of the Computer class and to access its property computer.db.

```javascript
const { computer } = new Computer({ seed, chain, network })
const { db } = computer
```

## db.put()
The ``db.put()`` method inputs an array of JSON objects and stores them in a transaction. Each element of the array is stored in a separate output. The method returns the array of locations of the outputs created.

```javascript
const data = [{a: 1}, {b: { c: 2 }}]
const locs = await computer.db.put(data)
// locs === ['0322...8dfe:0', '0322...8dfe:1']
```

## db.get()

The ``db.get()`` method returns the JSON objects stored at a given array of locations.

```javascript
const locs = await computer.db.put(data)
const fromChain = await computer.db.get(locs)
// fromChain === data
```

## db.update()

The ``db.update()`` method has two parameters: a list of locations and a list of JSON objects. It broadcasts a transaction that spends the locations and that has one output for each JSON object.

```javascript
const locs1 = await computer.db.put([{ n: 1 }])
const locs2 = await computer.db.update(locs1, [{ n: 2 }])
const fromChain = await computer.db.get(locs2)
// fromChain === [{n: 2}]
```

You can use db.get() to inspect the smart contract protocol. Try to call db.get()with the ids or rev of a smart object to see the data on the blockchain. Remember to pass the id inside an array likedb.get([a._id]).

# Wallet

We recommend creating a Wallet instance by creating a Computer instance and accessing the nested wallet property.

```javascript
const { computer } = new Computer({ seed, chain, network })
const { wallet } = computer.db
```

The wallet built into Bitcoin Computer is compatible with the widely used Bitcore library. This makes it easy to integrate Bitcoin Computer into existing apps.

## wallet.getMenmonic()

The ``wallet.getMnemonic()`` method returns a Bitcore compatible Javascript object encoding a Mnemonic. To return the mnemonic string call the toString() method.

```javascript
const mnemonic = wallet.getMnemonic()
const mnemonicString = mnemonic.toString()
```

## wallet.getPublicKey()

The ``wallet.getPublicKey()`` method returns a Bitcore compatible Javascript object encoding a public key. To return the public key in string encoding call the toString() method.

```javascript
const publicKey = wallet.getPublicKey()
const publicKeyString = publicKey.toString()
```

## wallet.getAddress()

The ``wallet.getAddress()`` method returns a Bitcore compatible Javascript object encoding a Bitcoin address. To return the address in string encoding call the toString() method.

```javascript
const address = wallet.getAddress()
const addressString = address.toString()
```

## wallet.getBalance()

The ``wallet.getBalance()`` method returns the current balance in satoshi.

```javascript
const balance = await wallet.getBalance()
```

## wallet.send()

The ``wallet.send()`` method sends an amount of satoshi to an address.

```javascript
const amount = 100000 // in satoshi
const address = '1FFsHfDBEh57BB1nkeuKAk25H44U7mmMXd'
const balance = await wallet.send(amount, address)
```
