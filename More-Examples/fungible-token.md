---
order: -30
---

# Fungible Token

We explore how an ERC20 style contracts (as defined in [EIP20](https://eips.ethereum.org/EIPS/eip-20)) can be built on Bitcoin. We use two classes, the ``TokenBag`` class describes a simple object that stores a number of tokens with a send function. The ``ERC20`` class makes it possible to deal with multiple token bags in the same way that a wallet deals with multiple unspent outputs.

## Token Bag

A ``TokenBag`` stores a number of tokens. It has a function ``send`` to send tokens to another user.

```js
export class TokenBag {
  constructor(to: PublicKey, supply: number) {
    this._owners = [to]
    this.tokens = supply
  }

  send(to: PublicKey, amount: number) {
    if (this.tokens < amount) throw new Error('Insufficient funds.')
    this.tokens -= amount
    return new TokenBag(to, amount)
  }
}
```

## Adding Approval

Approval is a mechanism in of the [ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol) contract where a user called ``spender`` is granted the right to spend certain number of tokens of the original owner.

We add two properties ``approvals`` and ``originalOwner`` to the ``TokenBag`` class and two methods ``approve`` and ``disapprove`` to grant and revoke approval. For now these methods work on an all or nothing bases for all tokens in the bag. We also add two propertied ``name`` and ``symbol`` according to the ERC20 standard.

```js #5-8,11-25
export class TokenBag {
  constructor(to: PublicKey, supply: number, name: string, symbol: string) {
    this._owners = [to]
    this.tokens = supply
    this.approvals = []
    this.originalOwner = to
    this.name = name
    this.symbol = symbol
  }

  approve(spender: PublicKey) {
    if(this.originalOwners.includes(spender))
      throw new Error('Cannot approve original owner.')

    this.approvals.push(spender)
    this._owners.push(spender)
  }

  disapprove(spender: PublicKey, amount: number) {
    if(this.originalOwners.includes(spender))
      throw new Error('Cannot disapprove original owner.')

    this._owners = this._owners.filter(owner => owner !== spender)
    this.approvals = this.approvals.filter(owner => owner !== spender)
  }

  send(to: PublicKey, amount: number) {
    if (this.tokens < amount) throw new Error()
    this.tokens -= amount
    return new ApprovableTokenBag(to, amount)
  }
}
```

## BRC20

In general each user will own several token bags, in the same way that users generally own multiple unspent outputs. The ``BRC20`` class adds functionality for sending tokens from multiple bags and for computing their balance across multiple bags. This is similar to the functionality of a traditional Bitcoin [wallet](wallet.md).

```js
import { Computer } from 'bitcoin-computer-lib'

export class BRC20 {
  constructor(name: string, symbol: string, mintId?: string, mnemonic?: string) {
    this.name = name
    this.symbol = symbol
    this.computer = new Computer({ mnemonic })
    this.mintId = mintId
  }

  async totalSupply(): Promise<number> {
   const rootBag = await this.computer.sync(this.mintId)
   return rootBag.tokens
  }

  async mint(publicKey: string, amount: number) {
    const args = [publicKey, amount, this.name, this.symbol]
    const tokenBag = await this.computer.new(TokenBag, args)
    this.mintId = tokenBag._root
  }

  async getBags(publicKey) {
    if(!this.mintId) throw new Error('MintId is undefined.')
    const revs = await this.computer.queryRevs({
      root: this.mintId,
      publicKey
    })
    return Promise.all(revs.map(rev => this.computer.sync(rev)))
  }

  async balanceOf(publicKey: string): Promise<number> {
    const tokenBags = await this.getTokenBags(publicKey)
    return tokenBags.reduce((prev, curr) => prev + curr.tokens, 0)
  }

  async transfer(to: string, amount: number) {
    const owner = this.computer.getPublicKey()
    const tokenBags = await this.getTokenBags(owner)
    while(amount > 0) {
      const bag = tokenBags.splice(0, 1)
      const available = Math.min(amount, bag.tokens)
      await bag.send(available, to)
      amount -= bag.tokens
    }
  }
}
```

We note that it is not necessary to check a message sender

This is a work in progress and some functions are missing. There is also a partial implementation on [Github](https://github.com/bitcoin-computer/BRC20).
