---
order: 100
---

# getNetwork

Returns the network.

### Syntax
```js
const network = computer.getNetwork()
```

### Types
```ts
() => string
```

### Return value

Returns a string encoding the network.

### Examples
```ts
const network = computer.getNetwork()
expect(network === 'regtest')
```