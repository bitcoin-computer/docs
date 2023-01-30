---
order: -40
---

# API Reference

In this section, we will describe the API of the class ```Computer```. Objects of this class can create and synchronize to smart objects; it also provides the usual methods of a wallet.

<table>
  <tr>
    <th>&nbsp;</th>
    <th>Method</th>
    <th>Description</th>
  </tr>

  <tr>
    <td rowspan="4">Basic</td>
    <td><a href="#constructor">constructor</a></td>
    <td>Creates a Computer object</td>
  </tr>

  <tr>
    <td><a href="#new">new</a></td>
    <td>Creates a new smart object</td>
  </tr>

  <tr>
    <td><a href="#query">query</a></td>
    <td>Find the latest revisions of smart objects with specific properties</td>
  </tr>

  <tr>
    <td><a href="#sync">sync</a></td>
    <td>Compute the state of a smart object at a specific revision</td>
  </tr>


  <tr>
    <td rowspan="4">Advanced</td>
    <td><a href="#encode">encode</a></td>
    <td>Encodes a JS expression in a transaction</td>
  </tr>

  <tr>
    <td><a href="#decode">decode</a></td>
    <td>Decodes a transaction into a JS expression</td>
  </tr>

  <tr>
    <td><a href="#encodenew">encodeNew</a></td>
    <td>Encodes a constructor call</td>
  </tr>

  <tr>
    <td><a href="#encodecall">encodeCall</a></td>
    <td>Encodes a function call</td>
  </tr>



  <tr>
    <td rowspan="2">Modules</td>
    <td><a href="#export">export</a></td>
    <td>Stores a module on the blockchain</td>
  </tr>

  <tr>
    <td><a href="#import">import</a></td>
    <td>Import a module from the blockchain</td>
  </tr>


  <tr>
    <td rowspan="14">Wallet</td>
    <td><a href="#fund">fund</a></td>
    <td>Funds a Bitcoin transaction</td>
  </tr>

  <tr>
    <td><a href="#sign">sign</a></td>
    <td>Signs a Bitcoin transaction</td>
  </tr>

  <tr>
    <td><a href="#broadcast">broadcast</a></td>
    <td>Broadcasts a Bitcoin transaction to the Bitcoin mining network</td>
  </tr>

  <tr>
    <td><a href="#send">send</a></td>
    <td>Sends an amount of satoshis to an address</td>
  </tr>

  <tr>
    <td><a href="#getaddress">getAddress</a></td>
    <td>Returns a string encoding Bitcoin address</td>
  </tr>

  <tr>
    <td><a href="#getbalance">getBalance</a></td>
    <td>Returns the current balance in satoshi</td>
  </tr>

  <tr>
    <td><a href="#getchain">getChain</a></td>
    <td>Returns the target blockchain</td>
  </tr>

  <tr>
    <td><a href="#getmenmonic">getMnemonic</a></td>
    <td>Returns a string encoding a BIP93 mnemonic sentence</td>
  </tr>

  <tr>
    <td><a href="#getnetwork">getNetwork</a></td>
    <td>Returns the target network</td>
  </tr>

  <tr>
    <td><a href="#getpassphrase">getPassphrase</a></td>
    <td>Returns the passphrase associated to the wallet</td>
  </tr>

  <tr>
    <td><a href="#getprivatekey">getPrivateKey</a></td>
    <td>Returns a string encoding a private key</td>
  </tr>

  <tr>
    <td><a href="#getpublickey">getPublicKey</a></td>
    <td>Returns a string encoding a public key</td>
  </tr>

  <tr>
    <td><a href="#getUtxos">getUtxos</a></td>
    <td>Returns an array of unspent transaction outputs</td>
  </tr>
</table>
<br />

## Basic

### Constructor

The constructor of the ```Computer``` class creates a new Bitcoin Computer wallet.

||| Example
```ts
import { Computer } from '@bitcoin-computer/lib'

const computer1 = new Computer()
const computer2 = new Computer({ chain: 'DOGE' })
const computer3 = new Computer({
  mnemonic: 'replace this seed'
  chain: 'LTC'
  network: 'regtest',
  path: "m/44'/0'/0'/0",
  url: 'https://my-ltc-node.com',
})
```
||| Type
```ts
new (params: {
  // Target blockchain, defaults to 'LTC'
  chain?: 'LTC' | 'BTC' | 'DOGE',

  // Network defaults to 'testnet'
  network?: 'mainnet' | 'testnet' | 'regtest',

  // BIP39 mnemonic phrase, defaults to a random phrase
  mnemonic?: string,

  // BIP32 path, defaults to "m/44'/0'/0'/0"
  path?: string,

  // BIP32 passphrase, defaults to ''
  passphrase?: string

  // Url of a Bitcoin Computer Node, defaults to https://node.bitcoincomputer.io
  url?: string,
} = {}) => Computer
```
|||
<br />

### new()

This method creates new smart objects. The inputs include a class and a list of arguments for the constructor of the class. The arguments can be of basic data type or smart objects.

||| Example
```ts
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
||| Type
```ts
<T extends new (...args: any) => any>(
  // A Javascript class
  constructor: T,

  // Arguments to the constructor
  args?: ConstructorParameters<T>,

  // A module specifier
  mod: string
) => Promise<InstanceType<T> & Location>;
```
|||
<br />

### query()

Returns an array containing the latest revisions that satisfy certain conditions, as specified in the parameter. For example, one can obtain all revisions owned by a public key or all revisions of a specific smart contract.

When a key is omitted, the condition is ignored. For example, if only ``class`` is set in the ```contract```parameter, the call will return all revisions of that class regardless of the owners.

||| Examples
```ts
class A extends Contract { ... }

// Return all revisions of a class
const revs1 = await computer.query({ contract: { class: A }})

// Return all revisions of a class that are owned by public key
const revs2 = await computer.query({ publicKey: '...', contract: { class: A }})

// Return 10 revisions of class A
const revs3 = await computer.query({ limit: 10, contract: { class: A }})

// Return all revisions of class A sorted by age
const revs4 = await computer.query({ order: 'ASC', contract: { class: A }})

// Return the latest revision of smart object with a specific id
const revs5 = await computer.query({ ids: ['...'] })
```
||| Type
```ts
(query: {
  // Return only revisions currently owned by a public key
  publicKey?: string

  // Return only revisions of smart object from a class
  contract?: <T extends new (...args: any) => any>{
    class: T,
    args?: ConstructorParameters<T>;
  }

  // Order results
  order?: 'ASC' | 'DEC',

  // Return only limited number of revisions
  limit?: number,

  // Return results starting from offset
  offset?: number,
}) => Promise<string[]>
```
|||
<br />

### sync()

This returns the smart object stored at a given revision.

||| Example
```ts
// Compute smart object from revision
const synced = await computer.sync(a._rev)

// Evaluates to true if "a" is a smart object
expect(synced).to.deep.equal(a)
```
||| Type
```ts
<T extends new (...args: any) => any>(
  rev: string
) => Promise<T>;
```
|||
<br />



## Advanced

### encode()

Encodes an expression, an environment and a module specifier into a Bitcoin transaction that can be broadcast to the Bitcoin mining network. The ``transaction`` object that is returned is compatible with the [Bitcore Library](https://www.npmjs.com/package/bitcore-lib).

||| Example
```ts
class C extends Contract {}
const computer = new Computer()
const transition = { exp: `${C} new ${C.name}()` }
const transaction = await computer.encode(transition)
expect(transaction).to.deep.equal({
  inputs: [],
  outputs: [Output, Output, Output],
  version: 2,
  nLockTime: 0,
  _feePerKb: 10000
})

```
||| Type
```ts
(transition: {
  exp: string,
  env?: { [s: string]: string },
  mod?: string
}) => Promise<Bitcore.Transaction>
```
|||

### decode()

Converts a Bitcore transaction into a transition object. The inverse of ``encode``.

||| Example
```ts
class C extends Contract {}
const computer = new Computer()
const transition = {
  exp: `${C} new ${C.name}()`,
  env: {},
  mod: ''
}
const transaction = await computer.encode(transition)
const decoded = await computer.decode(transaction)
expect(decoded).to.deep.equal(transition)
```
||| Type
```ts
(tx: Bitcore.Transaction) => Promise<{
  exp: string,
  env?: { [s: string]: string },
  mod?: string
}>
```
|||

### encodeNew()

Encodes a smart object creation into a Bitcoin transaction.

||| Example
```ts
class C extends Contract { }
const computer = new Computer()
const transaction = await computer.encodeNew({ constructor: C })
const decoded = await computer.decode(transaction)
expect(decoded).to.deep.eq({
  exp: `${C} new ${C.name}()`,
  env: {},
  mod: ''
})
```
||| Type
```ts
<T extends new (...args: any) => any>({
  constructor: T,
  args?: ConstructorParameters<T>,
  mod?: string
}) => Promise<Bitcore.Transaction>
```
|||

### encodeCall()

Encodes a smart object call into a Bitcoin transaction that can be broadcasted to the Bitcoin mining network.

||| Example
```ts
class Counter extends Contract {
  n: number
  constructor() {
    super()
    this.n = 0
  }
  inc(m) {
    this.n += m
  }
}
const computer = new Computer(confs.pop())
const counter = await computer.new(Counter)
const transaction = await computer.encodeCall({
  target: counter,
  property: 'inc',
  args: [1]
})
const decoded = await computer.decode(transaction)
expect(decoded).to.deep.eq({
  exp: `__bc__.inc(1)`,
  env: { __bc__: counter._rev },
  mod: ''
})
```
||| Type
```ts
({
  target: InstanceType<T> & Location,
  property: string,
  args: Parameters<InstanceType<T>[K]>,
  mod?: string
}) => Promise<Bitcore.Transaction>
```
|||


## Modules

### export

To optimize blockchain storage and reduce transaction costs, consider exporting your modules. This separates the modules from your smart objects, enabling the code for a class to be stored just once. Later in the text, we'll explain the process of funding, signing, and broadcasting the transaction.

||| Example
```ts
const revA = await computer.export(
  `export class A extends Contract {}`
)

const revB = await computer.export(`
  import { A } from '${revA}'
  export class B extends A {}
`)

const transition = { exp: `new B()`, mod: revB }
const transaction = await computer.encode(transition)
```
||| Type
```ts
(module: string) => Promise<string>
```
|||

### import

Imports a smart contract from a module specifier.

||| Example
```ts
class C extends Contract {}
const module = `export ${C}`
const computer = new Computer(confs.pop())
const rev = await computer.export(module)
const { C: CI } = await computer.import(rev)
expect(CI).to.equal(C)
```
||| Type
```ts
(name: string, rev: string) => Promise<new (...args: any) => any>;
```
|||

## Wallet

### fund

Funds a Bitcoin transaction with UTXOs from the wallet.

||| Example
```ts
class C extends Contract {}
const exp = `${C} new ${C.name}()`

const tx: Mnemonic.bitcore.transaction = await computer.encode({ exp })
expect(tx).not.to.be.undefined

await computer.fund(tx)
await computer.sign(tx)
const hex = await computer.broadcast(tx)
expect(hex).to.be.a('string')
```
||| Type
```ts
(tx: Mnemonic.bitcoin.Transaction): Promise<void>
```
|||

### sign

Signs a Bitcore transaction with the private key of the wallet.

||| Example
```ts
class C extends Contract {}
const exp = `${C} new ${C.name}()`

const tx: Mnemonic.bitcore.transaction = await computer.encode({ exp })
expect(tx).not.to.be.undefined

await computer.fund(tx)
await computer.sign(tx)
const hex = await computer.broadcast(tx)
expect(hex).to.be.a('string')
```
||| Type
```ts
(tx: Bitcore.Transaction) => void
```
|||

### broadcast

Broadcasts a Bitcoin transaction to the Bitcoin mining network.

||| Example
```ts
class C extends Contract {}
const exp = `${C} new ${C.name}()`

const tx: Mnemonic.bitcore.transaction = await computer.encode({ exp })
expect(tx).not.to.be.undefined

await computer.fund(tx)
await computer.sign(tx)
const hex = await computer.broadcast(tx)
expect(hex).to.be.a('string')
```
||| Type
```ts
(tx: Bitcore.Transaction) => Promise<string>
```
|||

### send

Sends an amount of satoshis to an address.

||| Example
```ts
const satoshi = 100000
const address = '1FFsHfDBEh57BB1nkeuKAk25H44U7mmMXd'
const balance = await wallet.send(satoshi, address)
```
||| Type
```ts
(amount: number, address: string) => Promise&lt;string&gt;
```
|||

### rpcCall

Calls a Bitcoin RPC method with the given parameters.

||| Example
```ts
await computer.rpcCall('getBlockchainInfo', '')
```
||| Type
```ts
(method: string, params: string) => Promise<any>
```
|||

### getAddress

Returns a string encoding Bitcoin address.

||| Example
```ts
const address = computer.getAddress()
```
||| Type
```ts
() => string
```
|||

### getBalance

Returns the current balance in satoshi.

||| Example
```ts
const balance = await computer.getBalance()
```
||| Type
```ts
() => Promise&lt;number&gt;
```
|||

### getPublicKey

Returns a string encoding a public key.

||| Example
```ts
const publicKey = computer.getPublicKey()
```
||| Type
```ts
() => string
```
|||

### getChain

Returns the chain of the ``computer`` object.

||| Example
```ts
const chain = computer.getChain()
```
||| Type
```ts
() => string
```
|||

### getNetwork

Returns the network of the ``computer`` object.

||| Example
```ts
const network = computer.getNetwork()
```
||| Type
```ts
() => string
```
|||

### getMenmonic

Returns a string encoding a BIP32 mnemonic sentence of the ``computer`` object.

||| Exmaple
```ts
const mnemonic = computer.getMnemonic()
```
||| Type
```ts
() => string
```
|||

### getPassphrase

Returns the passphrase of the ``computer`` object.

||| Example
```ts
const passphrase = computer.getPassphrase()
```
||| Type
```ts
() => string
```
|||

### getPrivateKey

Returns a string encoding a private key.

||| Example
```ts
const privateKey = computer.getPrivateKey()
```
||| Type
```ts
() => string
```
|||

### getPublicKey

Returns a string encoding a public key.

||| Example
```ts
const publicKey = computer.getPublicKey()
```
||| Type
```ts
() => string
```
|||

### getUtxos

Returns an array of UTXOs.

||| Example
```ts
const utxos = await computer.getUtxos()
```
||| Type
```ts
() => Promise<Bitcore.Transaction.UnspentOutput[]>
```
|||


# Appendix

## Overview

In the tables below we use the following abbreviations:

<pre>
type Location = { _id: string, _rev: string, _root: string }
</pre>
<br />
