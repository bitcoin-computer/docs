---
order: -30
---

# Chat

```js
class Chat {
  constructor(publicKey: string) {
    this.messages = []
    this._owners = [publicKey]
  }

  post(message: string) {
    this.messages.push(message)
  }

  invite(publicKey: string) {
    this._owners.push(publicKey)
  }
}
```
