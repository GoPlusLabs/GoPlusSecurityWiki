---
cover: ../../.gitbook/assets/Wintermute_Meta_1200_627.jpg
coverY: 0
---

# Wintermute & OP

## 摘要

| 状态       | 已警示                                                                          |   
| ------------ | ------------------------------------------------------------------------------ | 
| 类型         | 合约，跨链重放攻击                                            |   
| 日期         | Jun 05, 2022                                                                   |   
| 来源       | [@kelvinfichter](https://twitter.com/kelvinfichter/status/1534636743223386119) |   
| 直接损失  | 20M $OP tokens (\~ $30M)                                                       |   
| 项目地址 | -                                                                              |   

## 背景

Wintermute是知名的加密货币市商。他们当时正准备与Optimism合作，为$OP代币提供流动性。

Wintermute将其资产放置在不同链上的Gnosis Safe多签合约钱包中。

他们要求Optimism将$OP直接发送至Optimism链上他们的钱包地址。不过该地址仅在以太坊下部署链Gnosis Safe钱包，而Optimism上并没有，仅仅是个EOA。

## 攻击向量一条龙

### 整体布局

1. 找到在以太坊上的工厂部署交易，准备在OP上重放->
2. 在OP上使用该交易重放，部署出与以太坊上地址相同的工厂->
3. 通过工厂创建与以太坊上地址相同的钱包，黑客获胜

### 创建工厂的旧交易

为什么需要一笔旧的交易呢？因为当部署一个合约时，其地址是**确定性地**由`发送者地址`和`nonce`决定的。

> deployed address = keccak256(sender's address, sender's nonce)

如果其中任何一项不符合，则无法得到相应的地址。

工厂是由[0x1aa7451dd11b8cb16ac089ed7fe05efa00100a6a](https://etherscan.io/address/0x1aa7451dd11b8cb16ac089ed7fe05efa00100a6a)(Gnosis Deployer 3)在以太坊上部署的.

**工厂创建的交易**

[https://etherscan.io/tx/0x75a42f240d229518979199f56cd7c82e4fc1f1a20ad9a4864c635354b4a34261](https://etherscan.io/tx/0x75a42f240d229518979199f56cd7c82e4fc1f1a20ad9a4864c635354b4a34261)

**工厂地址**

[0x76e2cfc1f5fa8f6a5b3fc4c8f4788f0116861f9b](https://etherscan.io/address/0x76e2cfc1f5fa8f6a5b3fc4c8f4788f0116861f9b)

### 重放攻击

由于部署工厂的交易相对较老，并没有使用EIP155，该EIP通过在签名中包含chainId来防止重放攻击。

[攻击者](https://optimistic.etherscan.io/address/0x60b28637879b5a09d21b68040020ffbf7dba5107)注意到了这笔交易的特殊之处。他在以太坊上取得GD3的原始交易数据后，在Optimism上进行了重放。与该共计相关的有三个关键交易，攻击者将其全部重放。在此之前还向GD3发送了一些ETH作为gas。

**Gnosis Deployer 3在Optimism的交易**

[https://optimistic.etherscan.io/address/0x1aa7451dd11b8cb16ac089ed7fe05efa00100a6a](https://optimistic.etherscan.io/address/0x1aa7451dd11b8cb16ac089ed7fe05efa00100a6a)

![](../../.gitbook/assets/image.png)

上面的交易虽然看起来是来自GD3的，实际上是由黑客发起的。GD3本人甚至都没有注意到有人在冒充他。

### Nonce++

由于黑客已经创建了工厂，下一步就是生成目标地址的钱包。

**目标地址(Wintermute的钱包):**

\*\*\*\*[_0x4f3a120e72c76c22ae802d129f599bfdbc31cb8_](https://etherscan.io/address/0x4f3a120e72c76c22ae802d129f599bfdbc31cb81)

不巧的是, 这一步能发挥作用的原因也是因为，在创建合约时使用了`CREATE` 而非 `CREATE2`。

\
`CREATE` : `keccak256(rlp.encode(deployingAddress, nonce))[12:]`

`CREATE2` : `keccak256(0xff + deployingAddr + salt + keccak256(bytecode))[12:]`

在`CREATE2`中，有额外的参数来阻止不同的`msg.sender`生成相同的合约地址。

由此，黑客就可以在工厂中多次调用钱包生成函数，直至`nonce`达到目标值。

他还使用了一个包含for循环的助手合约，每次调用钱包创建时会循环162次。在60+次交易后，他达到了目标，将该EOA地址变成了一个受他控制的合约。

## 余波

来自[Oluwapelumi Adejumo](https://cryptoslate.com/optimism-hacker-confirms-they-are-whitehat-returns-most-of-stolen-funds/)的报道：

> 黑客会将剩余的200万$OP代币作为赏金。
>
> 代币返还的消息给OP币价带来了正面效果。在过去24小时内上升了13.2%。

## 总结

* 为了防止可能出现的各种不同的重放攻击，推荐使用EIP155交易标准以及`CREATE2`。
* EOA账户是天然跨链的，而智能合约钱包则不是。用户应该注意到这种差异。开发者或许可以尝试更健壮和用户友好的接口。
* 在事发前，自Wintermute发现问题后，有八天时间空挡。对于如此巨大的金额，他们的反应确实很慢。如果可以在第一时间报告给Optimism，或许可以找到解决方案。