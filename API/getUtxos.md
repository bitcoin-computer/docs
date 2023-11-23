---
order: 100
---

# getUtxos

Returns an array of unspent transaction outputs (UTXOs).

### Syntax
```js
const utxos = await computer.getUtxos()
```

### Types
```ts
() => Promise<BitcoinLib.Transaction.UnspentOutput[]>
```

### Return value

Returns an array of unspent transaction outputs (UTXOs).

### Examples
```ts
const utxos = await computer.getUtxos()
```