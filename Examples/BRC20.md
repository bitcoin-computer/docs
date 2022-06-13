---
order: -50
---

# BRC20

```js
import { Computer } from 'bitcoin-computer-lib'

export class BRC20 {
  constructor(name: string, symbol: string) {
    this.name = name
    this.symbol = symbol
    this.computer = new Computer({ mnemonic })
    this._owners = [this.computer.getPublicKey()]
  }

  async mint(publicKey: string, amount: number) {
    const owner = this.computer.getPublicKey()
    const tokenBag = await this.computer.new(TokenBag, [owner, amount])
    this.id = tokenBag._root
    return tokenBag
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

  async balanceOf(account: PublicKey): Promise<number> {
    const revs = await this.computer.queryRevs({ root: this.id, publicKey: account })
    const tokenBags = await Promise.all(revs.map(rev => this.computer.sync(rev)))

    return tokenBags.reduce((prev, curr) => prev + curr.tokens, 0)
  }

  async totalSupply(): Promise<number> {
   const rootBag = await this.computer.sync(this.id)
   return rootBag.tokens
  }
}

```
