---
order: -30
---

# Non Fungible Token (NFT)

Let's explore how ERC721 style contracts can be implemented on Bitcoin. The code is written and tested [here](https://github.com/bitcoin-computer/BRC721).

In the ```NFT``` class, we have a constructor and a ```send``` function. The constructor initializes a new token with a url and an initial owner. The ```send``` function adds the new owner to the list of owners of this token.

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

More information for how to build a non-fungible token application can be found [here](https://github.com/bitcoin-computer/non-fungible-token).


## BRC721

BRC721 is the ERC721 standard built on Bitcoin. It currently implements the ```IBRC721``` interface. Check out the code [here](https://github.com/bitcoin-computer/BRC721) for more information regarding the BRC721 token.

The ```BRC721``` class has the following functions: ```mint```, ```balanceOf```, ```ownerOf```, and ```transfer```.

The ```mint``` function mints a new non-fungible token. It first checks whether or not there is a ```masterNFT```. If not, and a name and a symbol are inputted, the computer generates a new NFT, which includes the name and symbol and a new owner's information, and sets it as this BRC721's ```masterNFT```. If there's already a ```masterNFT```, but its name does not match the name inputted, or the symbol does not match, throw an error. Otherwise, the ```masterNFT``` will mint a new token with the new owner's information.

```js
async mint(to: string, name?: string, symbol?: string): Promise<NFT> {
  if (!this.masterNFT && name && symbol) {
    this.masterNFT = this.computer.new(NFT, [to, name, symbol])
    return this.masterNFT
  }
  if (this.masterNFT.name !== name || this.masterNFT.symbol !== symbol) {
    throw new Error('Name or symbol mismatch when minting token.')
  }
  return this.masterNFT.mint(to)
}
```

The ```balanceOf``` function returns the number of NFTs the user owns. The ```ownerOf``` function returns the owner of a given token ID. The ```transfer``` function allows you to send the token to another user.

```js
async balanceOf(publicKey: string): Promise<number> {
  const revs = await this.computer.queryRevs({ publicKey, contract: NFT })
  const objects: NFT[] = await Promise.all(revs.map((rev) => this.computer.sync(rev)))
  objects.flatMap((object) => (object._root === this.masterNFT._root ? [object] : []))
  return objects.length
}

async ownerOf(tokenId: string): Promise<string[]> {
  const rev = await this.computer.idToRev(tokenId)
  const obj = await this.computer.sync(rev)
  return obj._owners
}

async transfer(to: string, tokenId: string) {
  const [rev] = await this.computer.getLatestRevs([tokenId])
  const obj = await this.computer.sync(rev)
  await obj.transfer(to)
}
```

!!!
The "Advanced Examples" Section is a work in progress. We are using the examples in this section to determine the final syntax and semantics for the Bitcoin Computer.
!!!