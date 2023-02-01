---
layout: page
description: This is a custom description for this page
---

![](/static/bitcoin-computer@1x.png)
# Bitcoin Computer

The Bitcoin Computer is a powerful on-chain L2 smart contract system that is designed specifically for UTXO-based blockchains.

Some of the key features include:

* Low fees through client-side validation
* The ability to write smart contracts in popular programming languages such as Javascript and Typescript
* Seamless integration into web applications
* Built-in privacy features such as off-chain storage and encryption
* Support for non-custodial applications
* Trustless and censorship-resistant design
* Compliance with ES6 module system
* Compatibility with both browser and node.js environments
* Support for Litecoin, with support for Bitcoin and Dogecoin coming soon

With the Bitcoin Computer, you can build a wide range of decentralized applications, such as fungible and non-fungible tokens, exchanges, games, office applications, social networks, messengers, AI-based applications, and much more.

## Getting Started

The best way to get started is to clone this repository and run one of the examples.

```bash
# download the code
git clone git@github.com:bitcoin-computer/monorepo.git

# install dependencies
cd monorepo
lerna bootstrap
```

## Examples

The examples are in the `packages` folder. To run an example have a look at the `README.md` file in the corresponding folder.

### Applications

* [Wallet](https://github.com/bitcoin-computer/monorepo/tree/main/packages/wallet): A minimal non-custodial wallet
* [Chat](https://github.com/bitcoin-computer/monorepo/tree/main/packages/chat): A p2p chat application where messages are communicated over the blockchain
* [Fungible Token](https://github.com/bitcoin-computer/monorepo/tree/main/packages/fungible-token): An application for minting, sending, and storing fungible tokens
* [Non Fungible Token](https://github.com/bitcoin-computer/monorepo/tree/main/packages/non-fungible-token): An application for minting, sending, and storing non-fungible tokens (NFTs)

### Standard Smart Contracts

* [BRC20 contract](https://github.com/bitcoin-computer/monorepo/tree/main/packages/BRC20): A implementation of the ERC20 Fungible Token Standard
* [BRC721 contract](https://github.com/bitcoin-computer/monorepo/tree/main/packages/BRC721): A implementation of the ERC721 Non-Fungible Token Standard

### Templates
* [Bitcoin Computer Node.js Template](https://github.com/bitcoin-computer/monorepo/tree/main/packages/node-js-boilerplate): A template for starting a new Bitcoin Computer application with [node.js](https://nodejs.org/en/)
* [Bitcoin Computer React Template](https://github.com/bitcoin-computer/monorepo/tree/main/packages/create-react-app-template): A template for starting a new Bitcoin Computer application with [Create React App](https://create-react-app.dev/)

## Core Library

The core smart contract library is based on two packages:

* [Bitcoin Computer Lib](https://www.npmjs.com/package/@bitcoin-computer/lib) A non-custodial web wallet for reading and writing smart contracts
* [Bitcoin Computer Node](https://github.com/bitcoin-computer/monorepo/tree/main/packages/node) Server side infrastructure for providing trustless access to smart contracts

## Price

It is free to develop and test a Bitcoin Computer application on testnet and regtest.

On Mainnet the fees for using the Bitcoin Computer are on average the same as the transaction fees charged by miners fees. For example, if the miner transaction fee is one cent, then you  pay two cent in total: one cent to the miners, and one cent to support the development of the Bitcoin Computer.

## Development Status

We have completed two internal audits and addressed any discovered issues. Currently, there are no known security vulnerabilities, however, it is possible that unknown vulnerabilities may exist. We plan to conduct one more thorough internal security audit prior to recommending the usage of the Bitcoin Computer in production environments.
## Getting Help

If you have any questions, please let us know in our <a href="https://t.me/thebitcoincomputer">Telegram group</a>, on <a href="https://twitter.com/TheBitcoinToken">Twitter</a>, or by email clemens@bitcoincomputer.io.

## License

[Bitcoin Computer Lib](https://www.npmjs.com/package/@bitcoin-computer/lib) and [Bitcoin Computer Node](https://github.com/bitcoin-computer/monorepo/tree/main/packages/node) are licensed under the [Creative Commons Attribution-NoDerivs 3.0 Unported License](https://creativecommons.org/licenses/by-nd/3.0/). All other [packages](https://github.com/bitcoin-computer/monorepo/tree/main/packages) in this monorepo are licensed under the [MIT License](https://opensource.org/licenses/MIT). See the individual packages for more information.
