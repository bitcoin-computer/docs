---
order: -10
---

# Getting Started

## Run the Tests

The easiest way to get started is to run the tests. If you get an error, have a look here.

```shell
git clone git@github.com:bitcoin-computer/bitcoin-computer-lib.git
cd bitcoin-computer-lib
yarn install
yarn test
```

## Run in Node.js

Create the file ```index.mjs```.

```javascript
import { Computer } from 'bitcoin-computer-lib'

// a smart contract
class Counter {
  constructor() {
    this.n = 0
  }

  inc() {
    this.n += 1
  }
}

// run the smart contract
;(async () => {
  // create Bitcoin Computer wallet
  const computer = new Computer({ seed: 'replace this seed' })

  // deploy a smart object
  const counter = await computer.new(Counter)

  // update a smart object
  await counter.inc()
  console.log(counter)
})()
```

Then, execute the following in the same directory.

```shell
yarn add bitcoin-computer-lib
node index.mjs
```

If you get an error ```Insufficient balance```, you have to fund the wallet; have a look [here](/troubleshoot.md). If the wallet is funded, you will see:

```javascript
Counter {
  n: 1,
  _id: '83553f27c9e4651323f1ebb...',
  _rev: '290923708ca56ea448dd67...'
}
```

You can clone the [boilerplate](https://github.com/bitcoin-computer/bitcoin-computer-node-js-boilerplate) or watch the [video walkthrough](https://www.youtube.com/watch?v=51ZFe_8mSPw).

## Run in the Browser

Create the file ```index.js```.

```javascript
import { Computer } from 'bitcoin-computer-lib'

class Counter {
  constructor() {
    this.n = 0
  }

  inc() {
    this.n += 1
  }
}

;(async () => {
  const computer = new Computer({ seed: 'replace this seed' })

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

Run the following in an empty directory, and open your browser at [http://localhost:1234](http://localhost:1234).

```shell
yarn add bitcoin-computer-lib
yarn global add parcel
parcel index.html
```

More information in this [video](https://www.youtube.com/watch?v=vcjzIFjt3VY).
