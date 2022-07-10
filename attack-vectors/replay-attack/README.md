---
cover: >-
  https://images.unsplash.com/photo-1494232410401-ad00d5433cfa?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHwxfHx0YXBlfGVufDB8fHx8MTY1NjQwNTI1MQ&ixlib=rb-1.2.1&q=80
coverY: 214.8803480783176
---

# 重放攻击

## 定义

在区块链领域,**重放攻击**是一种将旧交易的信息提交至新链、智能合约等目标上的攻击手法。

通常来说，在以太坊类似链上，有两种重放攻击：

* **交易签名重放：** 将一笔交易的[**原始签名**](./#获取交易原始签名)提交至一条链上。如果是同一条链则该该交易会失败，因为`nonce`不允许此类行为。而如果是跨链重放，则取决于原交易和目标链共识系统是否使用了包含`chainId`的抗跨链重放的EIP-155.
* **交易data重放:** 此处`data`就是一笔交易中的`data`字段。通过从其他交易中复制data，有可能对不严谨的身份/成员验证合约进行攻击.

在交易签名重放中，该交易看起来是由原交易发送者发送的，但实际上是由黑客发送的，而我们几乎没有任何手段能做区分。

## 其他细节

### 获取交易原始签名

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

#### 从Etherscan.io获取

某笔交易 -> `...` 图标 -> Get Raw Tx Hex.

注意：该结果为序列化后（RLP编码）的格式。需要用某些工具将其转换为人类可读格式
[https://flightwallet.github.io/decode-eth-tx/](https://flightwallet.github.io/decode-eth-tx/).

![](<../../.gitbook/assets/image (2).png>)
