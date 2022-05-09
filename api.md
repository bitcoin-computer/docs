---
order: -40
---

# Api

This Api describes the class Computer that can create and synchronize to smart objects, the class DB that can read, write, and update data on the blockchain, and the class Wallet, a Bitcoin wallet.

## Computer

The constructor of the Computer class takes one argument as shown below.

```javascript
const computer = new Computer({
  seed, // defaults to a random BIP39 mnemonic
  chain, // currently only 'LTC'
  network, // 'mainnet', 'testnet', or 'regtest', defaults to 'testnet'
  path // BIP32 path, defaults to "m/44'/0'/0'/0"
})
```

The constructor returns an object computer that contains a sub-object computer.db of class Dband a sub-object computer.db.wallet of class Wallet. In this section we explain the functions of the Computer class and we explain the Db and Wallet class in the following sections.
computer.new

The new() method creates new smart objects. It has two parameters, a class and a list of arguments. The arguments can be basic data types or smart objects.

```javascript
class A {
  constructor(n) {
    this.n = n
  }
}

const a1 = await computer.new(A, [1])
```

We encode an output as a string of the form <transaction-id>/<output-number>. We refer to an output encoded in this form as a location.

The new() method returns a smart object generated from the class and the arguments. It broadcasts a transaction that records the creation of the smart object. The smart object has the methods and properties defined in the class, and four extra properties:

 * _id: the location where the object is deployed. The id remains fixed throughout the lifecycle of the object.
 * _rev: the location where the object is currently stored. Initially _id and _rev are identical. When a method of the smart object is called its new state is stored at a new location, which is stored in the _rev property.
 * _owners: an array containing string encoded public keys. A function can only be called on a smart object if it was created by a computer instance that contains a private key corresponding to a public keys in the list of owners.
 * _amount: a number encoding the number of satoshis stored in the smart object.

The properties _id and _rev are read-only but the properties _owners and _amount can be assigned in constructor and function calls.

### computer.sync

The sync() method returns the smart object stored at a given revision.

```javascript
const revision = a1._rev
const a2 = await computer.sync(revision)
```

After running the code above a1 and a2 will be distinct objects with identical values.

### computer.getRevs

The getRevs() method returns an array containing the latest revisions owned by a given public key. The public keys are string encoded. If no parameter is passed to getRevs() the public key of the computer object is used.

```javascript
const publicKey = '03223d34686d6f19d20519156a030f7216e5d5bd6daa9442572bbaa446d06c8dfe'
const revs = await computer.getRevs(publicKey)
```

### computer.getLatestRev

The getLatestRev() method inputs an id and returns the latest revision of the smart object with that id. If no smart object with that id exists an error is thrown.

```javascript
const id = a1._id
const rev = await computer.getLatestRev(id)
```

After running the above code rev will be equal to a._rev if a1 is a smart object.

## Db

The recommended way to create an instance of the Db class is to create an object of the Computer class and to access its property computer.db.

```javascript
const computer = new Computer({ seed, chain, network })
const { db } = computer
```

### db.put
The put() method inputs an array of JSON objects and stores them in a transaction. Each element of the array is stored in a separate output. The method returns the array of locations of the outputs created.

```javascript
const data = [{a: 1}, {b: { c: 2 }}]
const locs = await computer.db.put(data)
// locs === ['0322...8dfe:0', '0322...8dfe:1']
```

### db.get

The get() method returns the JSON objects stored at a given array of locations.

```javascript
const locs = await computer.db.put(data)
const fromChain = await computer.db.get(locs)
// fromChain === data
```

### db.update

The update() method has two parameters: a list of locations and a list of JSON objects. It broadcasts a transaction that spends the locations and that has one output for each JSON object.

```javascript
const locs1 = await computer.db.put([{ n: 1 }])
const locs2 = await computer.db.update(locs1, [{ n: 2 }])
const fromChain = await computer.db.get(locs2)
// fromChain === [{n: 2}]
```

You can use db.get() to inspect the smart contract protocol. Try to call db.get()with the ids or rev of a smart object to see the data on the blockchain. Remember to pass the id inside an array likedb.get([a._id]).

## Wallet

We recommend creating a Wallet instance by creating a Computer instance and accessing the nested wallet property.

```javascript
const { db } = new Computer({ seed, chain, network })
const { wallet } = db
```

The wallet built into Bitcoin Computer is compatible with the widely used Bitcore library. This makes it easy to integrate Bitcoin Computer into existing apps.

### wallet.getMenmonic

The getMnemonic() method returns a Bitcore compatible Javascript object encoding a Mnemonic. To return the mnemonic string call the toString() method.

```javascript
const mnemonic = wallet.getMnemonic()
const mnemonicString = mnemonic.toString()
```

### wallet.getPublicKey

The getPublicKey() method returns a Bitcore compatible Javascript object encoding a public key. To return the public key in string encoding call the toString() method.

```javascript
const publicKey = wallet.getPublicKey()
const publicKeyString = publicKey.toString()
```

### wallet.getAddress

The getAddress() method returns a Bitcore compatible Javascript object encoding a Bitcoin address. To return the address in string encoding call the toString() method.

```javascript
const address = wallet.getAddress()
const addressString = address.toString()
```

### wallet.getBalance

The getBalance() method returns the current balance in satoshi.

```javascript
const balance = await wallet.getBalance()
```

### wallet.send

The send() method sends an amount of satoshi to an address.

```javascript
const amount = 100000 // in satoshi
const address = '1FFsHfDBEh57BB1nkeuKAk25H44U7mmMXd'
const balance = await wallet.send(amount, address)
```
