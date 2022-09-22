---
order: -10
---

# Chat

The Bitcoin Chat presented here is a hackable chat that runs on the blockchain. The chat does not need a database backend because all data is stored on the blockchain (currently runs on LTC testnet). The interface between the chat and the blockchain is provided by [Bitcoin Computer](https://www.bitcoincomputer.io).

The ```Chat``` class has a constructor and two functions, ```post``` and ```invite```.

The constructor initializes a owner of the chat and an empty ```messages``` array. The ```post``` function stores the new messages, and the ```invite``` function records the users who join the chat.

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

You can find the MIT licensed source code on [GitHub](https://github.com/bitcoin-computer/bitcoin-chat/blob/master/public/chat-sc.js).

This is intended as a starting point for developers to build on top of. Contributions are more than welcome. [Here](https://github.com/bitcoin-computer/bitcoin-chat) is the GitHub repository of Bitcoin Chat. Have a look at the open issues, or make a pull request with a new feature. You can also make a feature request by creating an issue.
