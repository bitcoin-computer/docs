---
layout: page
description: This is a custom description for this page
---

# Bitcoin Computer

Turing Complete smart contracts on Bitcoin

## Simple

Bitcoin Computer makes it possible to run smart contracts on Bitcoin. Smart contracts are Javascript classes and you can store instances of these classes (objects) on the Bitcoin blockchain. Each object has a unique location on the blockchain and if you know the location you can read the state of the object. The state can only be updated by function calls so that constraints on data updates can be enforced.

You can build  games, social networks, NFTs, fungible tokens, stable coins, exchanges, auctions, voting, office applications, artificial intelligence, every application you can think of. Building applications couldn't be simpler, just build a client side web application and whenever you want to store something in a database just use our client side [lib](https://www.npmjs.com/package/bitcoin-computer-lib) to store an object on the blockchain. Usually the entire backend can be replaced by a smart contract which can vastly simplify development.

## Efficient

On other blockchains fees are charged for every computational step and for every memory allocation. On Bitcoin there is no fee per computational step or memory allocation. This is because smart contracts are evaluated client side as opposed to by miners. Bitcoin Computer smart contracts are lightweight and do not impose any additional effort on nodes and miners. The only fee that needs to be paid is for the block space required to encode the name of the function being called and its parameters. This is mostly less than a few cents. This makes it possible, for the first time, to run compute and memory intense programs as smart contracts.


## Trustless

You can run your own [node](https://www.npmjs.com/package/bitcoin-computer-node). The node gives you trustless access to the Bitcoin Computer. Nobody can revoke your access if you have a copy of the node and the lib. You can think of a node as a universal backend that can power any application. You can run a node easily through docker.

We are launching on Litecoin. In the future we want to support all currencies in the Bitcoin family.
