---
order: -40
---

# API Reference

In this section, we will describe the API of the class ```Computer```. Objects of this class can create and synchronize to smart objects; it also provides the usual methods of a wallet.

In the tables below we use the following abbreviations:

```ts
type Class = new (...args: any) => any
type Transition = { exp: string, env?: { [s: string]: string }, mod?: string }
type Location = { _id: string, _rev: string, _root: string }
type Query = {
  publicKey?: string
  limit?: number,
  offset?: number,
  order?: string,
  contract?: {
    class: T,
    args?: ConstructorParameters<T>;
  }
}
```
<br />

## Smart Contract Interface

The following functions are sufficient for most smart contract development. Start here.

<table>
  <tr>
    <th>Method</th>
    <th>Description</th>
    <th>Type</th>
  </tr>
  <tr>
    <td><a href="#constructor">constructor</a></td>
    <td>Creates a Computer object</td>
    <td><pre style="font-size:12px">new (params: {<br>  mnemonic?: string,<br>  chain?: string,<br>  network?: string,<br>  path?: string, <br>  url?: string, <br>  passphrase?: string <br> }) => Computer</pre></td>
  </tr>
  <tr>
    <td><a href="#new">new</a></td>
    <td>Creates new smart objects</td>
    <td><pre style="font-size:12px">(constructor: T, args?: ConstructorParameters&lt;T&gt;, mod: string) => <br> Promise&lt;InstanceType&lt;T&gt; & Location&gt;</pre></td>
  </tr>
  <tr>
    <td><a href="#sync">sync</a></td>
    <td>Returns the smart object stored at a given revision</td>
    <td><pre style="font-size:12px">(rev: string) => Promise&lt;unknown&gt;</pre></td>
  </tr>
  <tr>
    <td><a href="#query">query</a></td>
    <td>Returns an array containing the latest revisions that satisfy certain conditions</td>
    <td><pre style="font-size:12px">(query: Query&lt;T&gt;) => Promise&lt;string[]&gt;</pre></td>
  </tr>
</table>
<br />

## Modules

You can use modules to decompose your smart contracts.

<table>
  <tr>
    <th>Method</th>
    <th>Description</th>
    <th>Type</th>
  </tr>

  <tr>
    <td><a href="#deploy">deploy</a></td>
    <td>Deploys an smart contract to the blockchain</td>
    <td><pre style="font-size:12px">(module: string) => Promise&lt;string&gt;</pre></td>
  </tr>

  <tr>
    <td><a href="#import">import</a></td>
    <td>Imports an ES6 module from the blockchain</td>
    <td><pre style="font-size:12px">(name: string, rev: string) => Promise&lt;Class&gt;</pre></td>
  </tr>

</table>
<br />

## Advanced Smart Contract Interface

If you want to use advanced features like off-chain signing or server side funding you can use the following functions.

<table>
  <tr>
    <th>Method</th>
    <th>Description</th>
    <th>Type</th>
  </tr>

  <tr>
    <td><a href="#decode">decode</a></td>
    <td>Converts a Bitcoin transaction into a transition object</td>
    <td><pre style="font-size:12px">(tx: Bitcore.Transaction) => Promise&lt;Transition&gt;</pre></td>
  </tr>

  <tr>
    <td><a href="#encode">encode</a></td>
    <td>Encodes an expression, an environment and a module specifier in a Bitcoin transaction</td>
    <td><pre style="font-size:12px">(transition: Transition) => Promise&lt;Bitcore.Transaction&gt;</pre></td>
  </tr>

  <tr>
    <td><a href="#encodenew">encodeNew</a></td>
    <td>Encodes a constructor call in a Bitcoin transaction</td>
    <td><pre style="font-size:12px">&lt;T extends Class&gt;({<br />  constructor: T,<br />  args?: ConstructorParameters&lt;T&gt;,<br />  mod?: string<br />}) => Promise&lt;Bitcore.Transaction&gt;</pre></td>
  </tr>

  <tr>
    <td><a href="#encodecall">encodeCall</a></td>
    <td>Encodes a function call into a Bitcoin transaction</td>
    <td><pre style="font-size:12px">({<br />  target: InstanceType&lt;T&gt; & Location,<br />  property: string,<br />  args: Parameters&lt;InstanceType&lt;T&gt;[K]&gt;,<br />  mod?: string<br />}) => Promise&lt;Bitcore.Transaction&gt;</pre></td>
  </tr>

</table>
<br />

## Wallet Interface

An object of class ``Computer`` also provides the functionality of a wallet.

<table>
  <tr>
    <th>Method</th>
    <th>Description</th>
    <th>Type</th>
  </tr>

  <tr>
    <td><a href="#broadcast">broadcast</a></td>
    <td>Broadcasts a Bitcoin transaction to the Bitcoin mining network</td>
    <td><pre style="font-size:12px">(tx: Bitcore.Transaction) => Promise&lt;string&gt;</pre></td>
  </tr>

  <tr>
    <td><a href="#fund">fund</a></td>
    <td>Funds a Bitcoin transaction</td>
    <td><pre style="font-size:12px">(tx: Bitcore.Transaction) => Promise&lt;void&gt;</pre></td>
  </tr>

  <tr>
    <td><a href="#getaddress">getAddress</a></td>
    <td>Returns a string encoding Bitcoin address</td>
    <td><pre style="font-size:12px">() => string</pre></td>
  </tr>

  <tr>
    <td><a href="#getbalance">getBalance</a></td>
    <td>Returns the current balance in satoshi</td>
    <td><pre style="font-size:12px">() => Promise&lt;number&gt;</pre></td>
  </tr>

  <tr>
    <td><a href="#getchain">getChain</a></td>
    <td>Returns the target blockchain</td>
    <td><pre style="font-size:12px">() => string</pre></td>
  </tr>

  <tr>
    <td><a href="#getmenmonic">getMnemonic</a></td>
    <td>Returns a string encoding a BIP93 mnemonic sentence</td>
    <td><pre style="font-size:12px">() => string</pre></td>
  </tr>

  <tr>
    <td><a href="#getnetwork">getNetwork</a></td>
    <td>Returns the target network</td>
    <td><pre style="font-size:12px">() => string</pre></td>
  </tr>

  <tr>
    <td><a href="#getpassphrase">getPassphrase</a></td>
    <td>Returns the passphrase associated to the wallet</td>
    <td><pre style="font-size:12px">() => string</pre></td>
  </tr>

  <tr>
    <td><a href="#getprivatekey">getPrivateKey</a></td>
    <td>Returns a string encoding a private key</td>
    <td><pre style="font-size:12px">() => string</pre></td>
  </tr>

  <tr>
    <td><a href="#getpublickey">getPublicKey</a></td>
    <td>Returns a string encoding a public key</td>
    <td><pre style="font-size:12px">() => string</pre></td>
  </tr>

  <tr>
    <td><a href="#getUtxos">getUtxos</a></td>
    <td>Returns an array of unspent transaction outputs</td>
    <td><pre style="font-size:12px">() => Promise&lt;Bitcore.Transaction.UnspentOutput[]&gt;</pre></td>
  </tr>

  <tr>
    <td><a href="#send">send</a></td>
    <td>Sends an amount of satoshis to an address</td>
    <td><pre style="font-size:12px">(amount: number, address: string) => Promise&lt;string&gt;</pre></td>
  </tr>

  <tr>
    <td><a href="#sign">sign</a></td>
    <td>Signs a Bitcoin transaction</td>
    <td><pre style="font-size:12px">(tx: Bitcore.Transaction) => void</pre></td>
  </tr>
</table>
<br />

## RPC interface

An object of class ``Computer`` can also access the RPC interface of the connected node.

<table>
  <tr>
    <th>Method</th>
    <th>Description</th>
    <th>Type</th>
  </tr>

  <tr>
    <td><a href="#rpccall">rpcCall</a></td>
    <td>Calls a Bitcoin RPC method</td>
    <td><pre style="font-size:12px">(method: string, params: string) => Promise&lt;any&gt;</pre></td>
  </tr>
</table>
<br />

## Details

### Constructor

The constructor of the ```Computer``` class creates a new Bitcoin Computer wallet.

```ts
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

### decode()

Converts a Bitcore transaction into a transition object.

```ts
const tx = new Bitcore.Transaction(txHex)
const transition = computer.decode(tx)
```

### deploy()

Deploys a smart contract to the blockchain, and returns the revision id.

```ts
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

```ts
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

```ts
class C extends Contract {}
const computer = new Computer()
const encoded = await computer.encodeNew({constructor: C})
const decoded = await computer.decode(encoded)
expect(decoded).to.eq({ exp: `new ${C.name}()`, env: {}, mod: '' })
```

### encodeCall()

Encodes a smart object call into a Bitcoin transaction that can be broadcasted to the Bitcoin mining network.

```ts
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

### getAddress()

Returns a string encoding Bitcoin address.

```ts
const address = computer.getAddress()
```

### getBalance()

Returns the current balance in satoshi.

```ts
const balance = await computer.getBalance()
```

### getChain()

Returns the chain of the ``computer`` object.

```ts
const chain = computer.getChain()
```

### getPublicKey()

Returns a string encoding a public key.

```ts
const publicKey = computer.getPublicKey()
```
### getMenmonic()

Returns a string encoding a BIP32 mnemonic sentence of the ``computer`` object.

```ts
const mnemonic = computer.getMnemonic()
```


### getNetwork()

Returns the network of the ``computer`` object.

```ts
const network = computer.getNetwork()
```

### getPassphrase()

Returns the passphrase of the ``computer`` object.

```ts
const passphrase = computer.getPassphrase()
```

### getPrivateKey()

Returns a string encoding a private key.

```ts
const privateKey = computer.getPrivateKey()
```

### getPublicKey()

Returns a string encoding a public key.

```ts
const publicKey = computer.getPublicKey()
```

### getUtxos()

Returns an array of UTXOs.

```ts
const utxos = await computer.getUtxos()
```

### import()

Imports a smart contract from a module specifier.

```ts
const fooClass = await computer.import('Foo','028e6db2f1c6eb0c449d89a4ce9a607029ede7d4')
```

### new()

This method creates new smart objects. The inputs include a class and a list of arguments for the constructor of the class. The arguments can be of basic data type or smart objects.

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

### query()

Returns an array containing the latest revisions that satisfy certain conditions, as specified in the parameter. For example, one can obtain all revisions owned by a public key or all revisions of a specific smart contract.

```ts
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

```ts
// Return all revisions of a class
const revs1 = await computer.query({ contract: { class: A }})
const revs2 = await computer.query({ publicKey: '...', contract: { class: A }})
const revs3 = await computer.query({ limit: 10, contract: { class: A }})
const revs4 = await computer.query({ order: 'ASC', contract: { class: A }})
```

 The ```query()``` function can be used to return the latest revision of the smart object with that id, or the latest revision of a smart object of a specific class.

```ts
// Get latest revision
const [rev] = await computer.query({ids:[a._id]})

// If a is up to date the next line evaluates to true
expect(rev).to.be(a._rev)

const revs5 = await computer.query({ ids: [a._id], contract: { class: A }})
```

### rpcCall()

Calls a Bitcoin RPC method with the given parameters.

```ts
await computer.rpcCall('getBlockchainInfo', '')
```

### send()

Sends an amount of satoshis to an address.

```ts
const satoshi = 100000
const address = '1FFsHfDBEh57BB1nkeuKAk25H44U7mmMXd'
const balance = await wallet.send(satoshi, address)
```

### sign()

Signs a Bitcore transaction with the private key of the wallet.

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
### sync()

This returns the smart object stored at a given revision.

```ts
// Compute smart object from revision
const synced = await computer.sync(a._rev)

// Evaluates to true if "a" is a smart object
expect(synced).to.deep.equal(a)
```
