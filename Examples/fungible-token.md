---
order: -30
---

# Fungible Token
## Fungible Token Bag

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

## Approvable Token Bag

```js
export class ApprovableTokenBag {
  constructor(owners: PublicKey[], supply: number) {
    this.tokens = supply
    this._owners = owners
    this.approvals = []
    this.originalOwners = owners
  }

  send(owners: PublicKey[], amount: number) {
    if (this.tokens < amount) throw new Error()
    this.tokens -= amount
    return new ApprovableTokenBag(owners, amount)
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

## BRC20

```js
import { Computer } from 'bitcoin-computer-lib'

export class BRC20 {
  constructor(name: string, symbol: string, mnemonic: string) {
    this.name = name
    this.symbol = symbol
    this.computer = new Computer({ mnemonic })
  }

  async mint(publicKey: string, amount: number) {
    const owner = this.computer.getPublicKey()
    const tokenBag = await this.computer.new(TokenBag, [owner, amount])
    this.id = tokenBag._root
  }

  async balanceOf(account: PublicKey): Promise<number> {
    const revs = await this.computer.queryRevs({ root: this.id, publicKey: account })
    const tokenBags = await Promise.all(revs.map(rev => this.computer.sync(rev)))

    return tokenBags.reduce((prev, curr) => prev + curr.tokens, 0)
  }

  transfer(to: PublicKey, amount: number) {
    const revs = await this.computer.queryRevs({ root: this.id, publicKey: to })
    const bags = await Promise.all(revs.map(rev => this.computer.sync(rev)))

    while(amount > 0) {
      const bag = bags.splice(0, 1)
      const available = Math.min(amount, bag.tokens)
      await bag.send(available, to)
      amount -= bag.tokens
    }
  }

  async totalSupply(): Promise<number> {
   const rootBag = await this.computer.sync(this.id)
   return rootBag.tokens
  }
}
```
