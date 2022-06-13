---
order: -20
---

# NFT

```js
class NFT {
  constructor(url: string, owner: string) {
    this.url = url
    this._owners = [owner]
  }

  send(to: string) {
    this._owners = [to]
  }
}
```
