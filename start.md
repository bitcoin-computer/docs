---
order: -10
---

# Start

The code consists of a client side library ([lib](https://www.npmjs.com/package/@bitcoin-computer/lib)) and server side library ([node](https://www.npmjs.com/package/bitcoin-computer)). By default, lib connects to an instance of a node we are running but you can run your own node and point lib to it. You can find examples in the [monorepo](https://github.com/bitcoin-computer/monorepo).

## Run the Tests

The easiest way to get started is to run the tests. If you get an error, have a look here.

```shell
git clone https://github.com/bitcoin-computer/monorepo.git
lerna bootstrap
```

## Run in Node.js

Create the file ```index.mjs```.

```javascript
import { Computer, Contract } from '@bitcoin-computer/lib'

// a smart contract
class Counter extends Contract {
  constructor() {
    super()
    this.n = 0
  }

  inc() {
    this.n += 1
  }
}

// run the smart contract
;(async () => {
  // create Litecoin or Bitcoin Computer wallet
  const computer = new Computer({ mnemonic: 'replace this seed' })

  // deploy a smart object
  const counter = await computer.new(Counter)

  // update a smart object
  await counter.inc()

  // log the state
  console.log(counter)
})()
```

Then, execute the following in the same directory.

```shell
yarn init -y
yarn add @bitcoin-computer/lib
node index.mjs
```

If you get an error ```Insufficient balance```, you have to fund the wallet; have a look [here](#troubleshooting). If the wallet is funded, you will see:

```javascript
Counter {
  n: 1,
  _id: '83553f27c9e4651323f1ebb...',
  _rev: '290923708ca56ea448dd67...',
  _root: '8136e4bceaf528ef6a8ff...'
}
```

You can clone the [boilerplate](@bitcoin-computer/node-js-boilerplate) or watch the [video walkthrough](https://www.youtube.com/watch?v=51ZFe_8mSPw).

## Run in the Browser

Create the file ```index.js```.

```javascript
import { Computer, Contract } from "https://unpkg.com/@bitcoin-computer/lib/dist/bc-lib.browser.min.mjs";

class Counter extends Contract {
  constructor() {
    super()
    this.n = 0
  }

  inc() {
    this.n += 1
  }
}

;(async () => {
  const computer = new Computer({ mnemonic: 'replace this seed' })

  const counter = await computer.new(Counter)
  document.getElementById("el").innerHTML = `Counter is ${counter.n}`

  await counter.inc()
  document.getElementById("el").innerHTML = `Counter is ${counter.n}`
})()
```

Create the file ```index.html```.

```html
<html>
  <body>
    <div id='el'></div>
    <script type="module" src="./index.js"></script>
  </body>
</html>
```

Run the following in an empty directory and open your browser at the defaults [http://localhost:8080](http://localhost:8080).

```shell
npm init -y
npm i http-server
http-server
```

See the readme file of the [Bitcoin Computer Monorepo](https://github.com/bitcoin-computer/monorepo).

## Troubleshooting

### Wallet is currently rescanning. Abort existing rescan or wait.

When you use a seed phrase for the first time, the Bitcoin node needs to re-scan the blockchain. This usually takes around 10 minutes but can take up to an hour. Please wait and try again.

### Insufficient balance in address

You need to fund the wallet inside the ```computer``` object.

The Bitcoin Computer currently supports Litecoin testnet. You can get free testnet coins from a Litecoin Testnet Faucet [here](https://testnet-faucet.com/ltc-testnet/), [here](https://testnet.help/en/ltcfaucet/testnet), [here](http://litecointf.salmen.website/), or [here](https://tltc.bitaps.com/).
We recommend generating a new seed phrase through a BIP39 generator.

## Getting Help

If you have any questions or are having trouble writing a smart contract, please get in touch. You can
 * ask a question in the [Telegram Group](https://t.me/joinchat/FMrjOUWRuUkNuIt7zJL8tg)
 * create and issue on [Github](https://github.com/bitcoin-computer/monorepo/issues)
 * get in touch on [Twitter](https://twitter.com/thebitcointoken)

We can also help you develop a smart contract on the Bitcoin Computer. Email ``clemens@bitcoincomputer.io`` for more information.
