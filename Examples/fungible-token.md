---
order: -30
---

# Fungible Token

```js
export class TokenBag {
  constructor(to: PublicKey, supply: number) {
    this.tokens = supply
    this._owners = [to]
  }

  send(to: PublicKey, amount: number) {
    if (this.tokens < amount) throw new Error()
    this.tokens -= amount
    return new TokenBag(to, amount)
  }
}
```

```js
export class ApprovableTokenBag {
  constructor(owners: PublicKey[], supply: number) {
    this.tokens = supply
    this._owners = owners
    this.approvals = []
  }

  send(owners: PublicKey[], amount: number) {
    if (this.tokens < amount) throw new Error()
    this.tokens -= amount
    return new ApprovableTokenBag(owners, amount)
  }

  get originalOwners(): string {
    return this._owners.filter(x => !this.approvals.includes(x))
  }

  approve(spender: PublicKey) {
    if(this.originalOwners.includes(spender)) throw new Error('Cannot approve original owner.')

    this.approvals.push(spender)
    this._owners.push(spender)
  }

  disapprove(spender: PublicKey) {
    if(this.originalOwners.includes(spender)) throw new Error('Cannot disapprove original owner.')

    this._owners = this._owners.filter(item => item !== spender)
    this.approvals = this.approvals.filter(item => item !== spender)
  }
}
```
