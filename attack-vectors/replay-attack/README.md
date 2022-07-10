---
cover: >-
  https://images.unsplash.com/photo-1494232410401-ad00d5433cfa?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHwxfHx0YXBlfGVufDB8fHx8MTY1NjQwNTI1MQ&ixlib=rb-1.2.1&q=80
coverY: 214.8803480783176
---

# Replay Attack

## Definition

In blockchain industry, a **Replay Attack** is an attack technique that acquires transaction info from old transactions and submits it to new chains, smart contracts or other targets.

Generally in Ethereum-like chains, there are two kinds of replay attacks:

* **Transaction signature replay:** Send the [**raw signature**](./#obtain-raw-signature-of-one-transaction) of one transaction to a chain. If it's the same chain as the original transaction, it won't work since there is `nonce` prohibiting this kind of behaviour. When it's a cross-chain transaction replay, it depends on whether the original transaction and target chain consensus have utilised EIP-155, which contains `chainId` to prevent cross-chain replay.
* **Transaction data replay:** Here `data` means exactly the `data` field in one transaction. By data copied from someone else, the attack could exploit a contract with awful membership/identity verification.

In transaction signature replay, the transaction looks like it was sent by the original sender, but actually it's the hacker who initiated the transaction, but we barely have measures to distinguish.

## Other Details

### Obtain raw signature of one transaction

#### Web3.js

```
web3.eth.getTransaction('0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b§234')
.then(console.log);

> {
    "hash": "0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b",
    "nonce": 2,
    "blockHash": "0xef95f2f1ed3ca60b048b4bf67cde2195961e0bba6f70bcbea9a2c4e133e34b46",
    "blockNumber": 3,
    "transactionIndex": 0,
    "from": "0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b",
    "to": "0x6295ee1b4f6dd65047762f924ecd367c17eabf8f",
    "value": '123450000000000000',
    "gas": 314159,
    "gasPrice": '2000000000000',
    "input": "0x57cb2fc4"
}
```

#### From Etherscan.io

Some transaction -> `...` Icon -> Get Raw Tx Hex.

Note: The result is in unserialised format(RLP encoded). You need to convert it to human readable format with some tools like [https://flightwallet.github.io/decode-eth-tx/](https://flightwallet.github.io/decode-eth-tx/).

![](<../../.gitbook/assets/image (2).png>)
