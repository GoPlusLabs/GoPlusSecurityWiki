---
cover: ../../../.gitbook/assets/polynetwork.jpg
coverY: 0
---

# Poly Network

## 摘要

Poly Network是个跨链协议。其中的守卫具有移动资金的能力。通过将守卫替换为自己，黑客盗取了巨额资产。

黑客利用了相关合约的下列漏洞：

* 可调用任意合约，没有足够的检查或限制
* `abi.encodePacked`组合的函数签名的哈希碰撞

| 状态       | 已修复                                                                                            |  
| ------------ | ------------------------------------------------------------------------------------------------ | 
| 类型         | 合约，跨链                                                                            |  
| 日期         | August 10, 2021                                                                                  |   
| 来源       | [Slowmist](https://slowmist.medium.com/the-root-cause-of-poly-network-being-hacked-ec2ee1b0c68f) |   
| 直接损失  | $610 million                                                                                     |   
| 项目仓库 | [https://github.com/polynetwork/eth-contracts](https://github.com/polynetwork/eth-contracts)     |   

## 合约结构

本事件中有两个相关合约：`EthCrossChainData(ECCD)`和`EthCrossChainManager(ECCM)`。

在正常情况下，守卫会监控从原链到目标链的跨链交易，然后向`ECCD`提交区块头，证明和其他数据。

然后，在`ECCM`中，跨链交易的有效性会被合约验证，如果检查通过（如，经过守卫签名，梅克尔树根正确，等）再执行**交易中指定的合约调用**。这些合约调用原本预期是用来执行跨链操作的，但其限制条件非常弱。

### EthCrossChainData(ECCD)

* 更新并存储所有的跨链交易。
* 添加，修改，存储所有的守卫的公钥。守卫具有移动资金和进行其他关键操作（如对跨链操作签名）的能力。

**putCurEpochConPubKeyBytes()**

```
    function putCurEpochConPubKeyBytes(
        bytes memory curEpochPkBytes)
        public whenNotPaused onlyOwner returns (bool) {
            ConKeepersPkBytes = curEpochPkBytes;
            return true;
    }
```

修改守卫的公钥。由于有`onlyOwner`修饰符，只能被`ECCD`的所有者`ECCM`调用。

### **EthCrossChainManager(ECCM)**

* 验证Poly Chain的区块头和证明，执行从Poly Chain到以太坊的跨链交易。
* `ECCD`的所有者。

#### **changeBookKeeper()**

```
function changeBookKeeper(
        bytes memory rawHeader,
        bytes memory pubKeyList,
        bytes memory sigList)
    whenNotPaused public returns(bool) {
```

该方法是正常且合法的替换守卫的方法。黑客无法调用该方法，因为该方法需要来自守卫的签名。

**crossChain()**

```
function crossChain(
        uint64 toChainId,
        bytes calldata toContract,
        bytes calldata method,
        bytes calldata txData)
     whenNotPaused external returns (bool) {
```

该方法用于从原链上提交一笔跨链交易。其信息会被守卫中继至目标链上的`ECCD`。

**verifyHeaderAndExecuteTx()**

```
function verifyHeaderAndExecuteTx(
        bytes memory proof,
        bytes memory rawHeader,
        bytes memory headerProof,
        bytes memory curRawHeader,
        bytes memory headerSig) {
```

验证Poly Chain的区块头，证明，以及守卫的签名，并执行跨链交易。

**executeCrossChainTx()**

```
function _executeCrossChainTx(
        address _toContract,
        bytes memory _method,
        bytes memory _args,
        bytes memory _fromContractAddr,
        uint64 _fromChainId)
    internal returns (bool){
```

一个internal函数，由**verifyHeaderAndExecuteTx**调用，并执行指定操作。

## 攻击向量和细节

### 整体布局

1. 黑客精心构建了替换守卫的参数
2. 黑客在其他链上调用了`crossChain()`并提交跨链交易，但这些并不是正常的跨链交易，其内容是替换守卫
3. Poly Network的守卫将这些交易中继至目标链上的`ECCD`
4. `ECCM`从`ECCD`中验证并执行了这些恶意跨链交易
5. 转移资金跑路，攻击完成

除了第一步，剩余的都是比较平常的操作，因此我们这里只解析第一步的细节，恶意参数。

### 恶意参数的构建

我们看下`_executeCrossChainTx()`是如何调用其他合约的：

```
    (success, returnData) = _toContract.call(
        abi.encodePacked(
            bytes4(keccak256(abi.encodePacked(_method, "(bytes,bytes,uint64)"))),
            abi.encode(_args, _fromContractAddr, _fromChainId)));
```

基本上，可以解读为`ContractAddress.call(functionSelector, paramters)`。

**函数选择子的哈希碰撞**

`functionSelector` = `bytes4(keccak256(abi.encodePacked(_method, "(bytes,bytes,uint64)")))`. 通过 `bytes4` 类型强转,只有前4字节得以保留，而Solidity的函数选择子也是4字节长。

不过看一下Solidity官方文档对`abi.encodePacked()`的说明：

> 如果你使用了`keccak256(abi.encodePacked(a, b))`且`a`和`b`都是动态类型，则非常容易通过将`a`和`b`的一部分内容移动到对方中去来产生哈希碰撞。具体来说，`abi.encodePacked("a", "bc") == abi.encodePacked("ab", "c")`。如果你为签名，真实性和数据完整性等使用了`abi.encodePacked`，则确保总是使用同一种类型，并检查其中至多有一项是动态的。除非有特殊原因，应优先考虑`abi.encode`。

也就是说，在`bytes4(keccak256(abi.encodePacked(_method, "(bytes,bytes,uint64)")))`中, `"(bytes,bytes,uint64)"`是一个非常弱的限制，因为`encodePacked`仅仅是将所有字符串接起来，且只有前4字节会作为keccak256后的函数选择子，黑客可以轻松地暴力运算出一个相同的函数选择子。

`putCurEpochConPubKeyBytes(bytes)`函数用来更新守卫的私钥，其选择子为`0x41973cd9`。你可以通过`ethers.utils.id ('putCurEpochConPubKeyBytes(bytes)').slice(0, 10)`或其他各种在线工具来计算。

黑客暴力破解了`RANDOM_STRING`来满足下列等式：

`'0x41973cd9' == ethers.utils.id ('RANDOM_STRING(bytes,bytes,uint64)').slice(0, 10)`.

有许多`RANDOM_STRING`可以满足这个式子，比如：

* `f1121318093`
* `func10487987874260605968`

**被省略的参数**

前面章节的代码块里的函数需要三个参数:

`abi.encode(_args, _fromContractAddr, _fromChainId)`

但在`putCurEpochConPubKeyBytes(bytes)`中只需要一个参数。那么将冗余的参数传递给函数，是合法调用吗？分情况，此处是yes，通过这种方式调用函数，这些多余的参数只会被忽略。

黑客将地址 `0xA87fB85A93Ca072Cd4e5F0D4f178Bc831Df8a00B`传入了`_args`，然后会被传至`putCurEpochConPubKeyBytes(bytes memory curEpochPkBytes)` 来替换守卫的地址。

参数构建完成。

## 总结

* **权限控制很重要**: 在复杂的工程中, `onlyOwner` 或其他形式的权限控制有可能会失效。可能会有其他攻击向量来访问到核心区域。开发者应该有一个更加整体的视野来巩固权限控制。
* **谨防哈希碰撞**: 能任意或有限制地调用函数是优秀的智能合约可扩展性的设计。 建议开发者放弃`call(bytes4(keccak256("f(uint256)")), a, b)` ，而使用`call(abi.encodeWithSignature("f(uint256)", a, b))`，并尽可能避免使用`abi.encodePacked()`。

## 余波

来自[Kudelski](https://research.kudelskisecurity.com)的报告：

> Poly Network asked the hacker to return the funds. The security company Slowmist published findings on the alleged hacker, claiming that the hacker’s identity had been exposed and that the group had access to the hacker’s email and IP address. According to Slowmist, the hacker was able to take advantage of a relatively unknown crypto exchange in Asia and they claimed to have a lot of information about the attacker.
>
> Whether this is true or not, the hacker started returning funds to Poly on Wednesday. By August 11th 15:00 UTC nearly half worth of tokens have been returned, and the hacker claims to be ready to return more in exchange for the unfreeze of the Tether tokens. A second message embedded in a transaction reads: **“IT’S ALREADY A LEGEND TO WIN SO MUCH FORTUNE. IT WILL BE AN ETERNAL LEGEND TO SAVE THE WORLD. I MADE THE DECISION, NO MORE DAO”**.
>
> While this story develops, it is not superfluous to remind that “blockchain” is not synonymous with “security”. It is very important to audit the security of your applications, including smart contracts.

## 参考

[https://docs.soliditylang.org/en/v0.8.15/abi-spec.html](https://docs.soliditylang.org/en/v0.8.15/abi-spec.html)

[https://research.kudelskisecurity.com/2021/08/12/the-poly-network-hack-explained/](https://research.kudelskisecurity.com/2021/08/12/the-poly-network-hack-explained/)

[https://slowmist.medium.com/the-root-cause-of-poly-network-being-hacked-ec2ee1b0c68f](https://slowmist.medium.com/the-root-cause-of-poly-network-being-hacked-ec2ee1b0c68f)
