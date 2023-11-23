---
order: 100
---

# broadcast

Broadcasts a Bitcoin transaction to the Bitcoin mining network.

### Syntax
```js
const txId = await computer.broadcast(tx)
```

### Types
```ts
(tx: BitcoinLib.Transaction) => Promise<string>
```

### Parameters

#### tx
A Bitcoin transaction object.


### Return value

If broadcast is successful, it returns an string encoding the transaction id. Otherwise, an error is thrown.

### Examples
```ts
class C extends Contract {}
const transition = { exp: `${C} new C()` }
const { tx } = await computer.encode(transition)
const txId = await computer.broadcast(tx)
```