---
order: -40
---

# API Reference

In this section, we will describe the API of the class ```Computer```. Objects of this class can create and synchronize to smart objects; it also provides the usual methods of a wallet.

## Smart Contract Interface

Property   | Description | Type
---    | --- | ---
[constructor](#constructor) | Creates a Computer object | (params: { <br> &nbsp;&nbsp; mnemonic?: string, <br> &nbsp;&nbsp; chain?: string, <br> &nbsp;&nbsp; network?: string, <br> &nbsp;&nbsp; path?: string, <br> &nbsp;&nbsp; url?: string, <br> &nbsp;&nbsp; passphrase?: string <br> }) => Computer
[broadcast](#broadcast) | Broadcasts a Bitcoin transaction to the Bitcoin mining network | (tx: BitcoreTx) => Promise\<string\>
[decode](#decode) | Converts a Bitcoin transaction into a transition object | (tx: BitcoreTx) => <br> Promise<Transition>
[deploy](#deploy) | Deploys an smart contract to the blockchain | (module: string) => Promise\<string\>
[encode](#encode)  | Encodes an expression, an environment and a module specifier into a Bitcoin transaction | (tr: Transition) => <br> Promise<BitcoreTx>
[encodeNew](#encodenew) | Encodes a smart object creation into a Bitcoin transaction | ({ constructor: T, args?: ConstructorParameters<T>, mod?: string }) => Promise\<BitcoreTx\>
[encodeCall](#encodecall) | Encodes a smart object call into a Bitcoin transaction | ({ target: InstanceType<T> & Location, property: string, args: Parameters<InstanceType<T>[K]>, mod?: string }) => Promise\<BitcoreTx\>
[fund](#fund) | Funds a Bitcoin transaction | (tx: BitcoreTx) => Promise\<void\>
[getAddress](#getaddress) | Returns a string encoding Bitcoin address | () => string
[getBalance](#getbalance) | Returns the current balance in satoshi | () => Promise\<number\>
[getChain](#getchain) | Returns the target blockchain | () => string
[getMnemonic](#getmenmonic) | Returns a string encoding a BIP93 mnemonic sentence | () => string
[getNetwork](#getnetwork) | Returns the target network | () => string
[getPassphrase](#getpassphrase) | Returns the passphrase associated to the wallet | () => string
[getPrivateKey](#getprivatekey) | Returns a string encoding a private key | () => string
[getPublicKey](#getpublickey) | Returns a string encoding a public key | () => string
[getUtxos](#getutxos) | Returns an array of unspent transaction outputs | () => Promise\<BitcoreUTXO[]\>
[import](#import) | Imports a smart contract from the blockchain | (name: string, rev: string) => Promise\<Class\>
[new](#new) | Creates new smart objects | (constructor: T, args?: ConstructorParameters\<T\>, mod: string) => <br> Promise\<InstanceType\<T\> & Location\>
[query](#query) | Returns an array containing the latest revisions that satisfy certain conditions | (query: Query<T>) => Promise<string[]>
[rpcCall](#rpccall) | Calls a Bitcoin RPC method  | (method: string, params: string) => Promise\<any\>
[sync](#sync) | Returns the smart object stored at a given revision | (rev: string) => Promise\<unknown\>
[send](#send) | Sends an amount of satoshis to an address | (amount: number, address: string) => Promise\<string\>
[sign](#sign) | Signs a Bitcoin transaction | (tx: BitcoreTx) => void

Types used in the interface:

Type | Description | Details
---    | --- | ---
BitcoreTx | A Bitcoin transaction | Menmonic.bitcore.Transaction
BitcoreUTXO | A Bitcoin Unspent Transaction Output | Menmonic.bitcore.Transaction.UnspentOutput
Class | A smart contract | new (...args: any) => any
Query | A query | { <br> &nbsp;&nbsp; publicKey?: string <br> &nbsp;&nbsp; limit?: number, <br> &nbsp;&nbsp; offset?: number, <br> &nbsp;&nbsp; order?: string, <br> &nbsp;&nbsp; contract?: { <br> &nbsp;&nbsp;&nbsp; class: T, <br> &nbsp;&nbsp;&nbsp; args?: ConstructorParameters\<T\> <br>&nbsp;&nbsp;} <br>}
Transition | A transition | { exp: string, env?: { [s: string]: string }, mod?: string }
Location | Key-value pairs that identify a smart object | { _id: string, _rev: string, _root: string }


### Constructor

The constructor of the ```Computer``` class creates a new Bitcoin Computer wallet.

```js
import { Computer } from '@bitcoin-computer/lib'

const computer = new Computer({
  // BIP39 mnemonic phrase, defaults to a random phrase
  mnemonic?: string, //

  // Target blockchain, defaults to 'LTC'
  chain?: string,

  // Network (one of 'mainnet', 'testnet', or 'regtest'),
  // defaults to 'testnet'
  network?: string,

  // BIP32 path, defaults to "m/44'/0'/0'/0"
  path?: string,

  // Url of a Bitcoin Computer Node, defaults to
  // https://node.bitcoincomputer.io
  url?: string,

  // The passphrase associated to the wallet, defaults to ''
  passphrase?: string

})
```

### broadcast()

Broadcasts a Bitcoin transaction to the Bitcoin mining network.

```js
class C extends Contract {}
const exp = `${C} new ${C.name}()`

const tx: Mnemonic.bitcore.transaction = await computer.encode({ exp })
expect(tx).not.to.be.undefined

await computer.fund(tx)
await computer.sign(tx)
const hex = await computer.broadcast(tx)
expect(hex).to.be.a('string')
```

### decode()

Converts a Bitcore transaction into a transition object.

```js
const tx = new Bitcore.Transaction(txHex)
const transition = computer.decode(tx)
```

### deploy()

Deploys a smart contract to the blockchain, and returns the revision id.

```js
class Vehicle extends Contract {
  constructor() {
    super()
  }
  drive() {
    return 'driving'
  }
  stop() {
    return 'stopped'
  }
}

class Car extends Vehicle {
  constructor() {
    super()
  }

  drive() {
    return 'driving a car'
  }
}
const revVehicle = await computer.deploy(`export ${Vehicle}`)
const car = await computer.new(Car, [], revVehicle)
expect(await car.drive()).eq('driving a car')
expect(await car.stop()).eq('stopped')
```

### encode()

Encodes an expression, an environment and a module specifier into a Bitcoin transaction that can be broadcasted to the Bitcoin mining network.

```js
class C extends Contract {}
const exp = `${C} new ${C.name}()`

const tx: Mnemonic.bitcore.transaction = await computer.encode({ exp })
await computer.wallet.fundTx(tx)
computer.wallet.signTx(tx)
const hex: string = tx.toString('hex')
const tx2: Tx = await computer.txFromHex({ hex })
expect(tx2.tx.toString('hex')).to.eq(hex)

```
### encodeNew()

Encodes a smart object creation into a Bitcoin transaction that can be broadcasted to the Bitcoin mining network.

```js
class C extends Contract {}
const computer = new Computer()
const encoded = await computer.encodeNew({constructor: C})
const decoded = await computer.decode(encoded)
expect(decoded).to.eq({ exp: `new ${C.name}()`, env: {}, mod: '' })
```

### encodeCall()

Encodes a smart object call into a Bitcoin transaction that can be broadcasted to the Bitcoin mining network.

```js
class C extends Contract {
  constructor() {
    super()
  }
  foo() {
    return 'bar'
  }
}
const computer = new Computer()
const encoded = await computer.encodeCall({constructor: C, property: 'foo'})
const decoded = await computer.decode(encoded)
expect(decoded).to.eq({ exp: `new ${C.name}().foo()`, env: {}, mod: '' })
```
### fund()

Funds a Bitcoin transaction with UTXOs from the wallet.

```js
class C extends Contract {}
const exp = `${C} new ${C.name}()`

const tx: Mnemonic.bitcore.transaction = await computer.encode({ exp })
expect(tx).not.to.be.undefined

await computer.fund(tx)
await computer.sign(tx)
const hex = await computer.broadcast(tx)
expect(hex).to.be.a('string')
```

### getAddress()

Returns a string encoding Bitcoin address.

```js
const address = computer.getAddress()
```

### getBalance()

Returns the current balance in satoshi.

```js
const balance = await computer.getBalance()
```

### getChain()

Returns the chain of the ``computer`` object.

```js
const chain = computer.getChain()
```

### getPublicKey()

Returns a string encoding a public key.

```js
const publicKey = computer.getPublicKey()
```
### getMenmonic()

Returns a string encoding a BIP32 mnemonic sentence of the ``computer`` object.

```js
const mnemonic = computer.getMnemonic()
```


### getNetwork()

Returns the network of the ``computer`` object.

```js
const network = computer.getNetwork()
```

### getPassphrase()

Returns the passphrase of the ``computer`` object.

```js
const passphrase = computer.getPassphrase()
```

### getPrivateKey()

Returns a string encoding a private key.

```js
const privateKey = computer.getPrivateKey()
```

### getPublicKey()

Returns a string encoding a public key.

```js
const publicKey = computer.getPublicKey()
```

### getUtxos()

Returns an array of UTXOs.

```js
const utxos = await computer.getUtxos()
```

### import()

Imports a smart contract from a module specifier.

```js
const fooClass = await computer.import('Foo','028e6db2f1c6eb0c449d89a4ce9a607029ede7d4')
```

### new()

This method creates new smart objects. The inputs include a class and a list of arguments for the constructor of the class. The arguments can be of basic data type or smart objects.

```js
// A smart contract
class A extends Contract {
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

### query()

Returns an array containing the latest revisions that satisfy certain conditions, as specified in the parameter. For example, one can obtain all revisions owned by a public key or all revisions of a specific smart contract.

```js
const revs = await computer.query({
  // Return only revisions owned by a public key
  publicKey: string,

  // Return only revisions of smart object from a class
  contract: { class: string, args: any[] },

  // Return only limited number of revisions
  limit: number,

  // Returns the results in ASC or DEC order
  order: 'ASC' | 'DEC',

  // Return only revisions with a specific id
  ids: string[],
  
})
```

When a key is omitted, the condition is ignored. For example, if only ``class`` is set in the ```contract```parameter, the call will return all revisions of that class regardless of the owners.

```js
// Return all revisions of a class
const revs1 = await computer.query({ contract: { class: 'A' }})
const revs2 = await computer.query({ publicKey: '...', contract: { class: 'A' }})
const revs3 = await computer.query({ limit: 10, contract: { class: 'A' }})
const revs4 = await computer.query({ order: 'ASC', contract: { class: 'A' }})
```

 The ```query()``` function can be used to return the latest revision of the smart object with that id, or the latest revision of a smart object of a specific class.

```js
// Get latest revision
const [rev] = await computer.query({ids:[a._id]})

// If a is up to date the next line evaluates to true
expect(rev).to.be(a._rev)

const revs5 = await computer.query({ ids: [a._id], contract: { class: 'A' }})
```

### rpcCall()

Calls a Bitcoin RPC method with the given parameters.

```js
await computer.rpcCall('getBlockchainInfo', '')
```

### send()

Sends an amount of satoshis to an address.

```js
const satoshi = 100000
const address = '1FFsHfDBEh57BB1nkeuKAk25H44U7mmMXd'
const balance = await wallet.send(satoshi, address)
```

### sign()

Signs a Bitcore transaction with the private key of the wallet.

```js
class C extends Contract {}
const exp = `${C} new ${C.name}()`

const tx: Mnemonic.bitcore.transaction = await computer.encode({ exp })
expect(tx).not.to.be.undefined

await computer.fund(tx)
await computer.sign(tx)
const hex = await computer.broadcast(tx)
expect(hex).to.be.a('string')
```
### sync()

This returns the smart object stored at a given revision.

```js
// Compute smart object from revision
const synced = await computer.sync(a._rev)

// Evaluates to true if "a" is a smart object
expect(synced).to.deep.equal(a)
```
