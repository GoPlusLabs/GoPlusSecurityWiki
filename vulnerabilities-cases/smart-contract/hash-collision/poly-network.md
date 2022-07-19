---
cover: ../../../.gitbook/assets/polynetwork.jpg
coverY: 0
---

# Poly Network

## Abstract

Poly Network is a cross-chain protocol. The hacker stole a huge amount of assets by replacing its Keepers, who have the power to move funds, with himself.

The following vulnerabilities of the related contracts were exploited:

* Ability to call arbitrary contract with insufficient checks or restrictions
* Hash collision of function signature with `abi.encodePacked`

| Status       | Fixed                                                                                            |
| ------------ | ------------------------------------------------------------------------------------------------ |
| Type         | Contract, Cross-Chain                                                                            |
| Date         | August 10, 2021                                                                                  |
| Source       | [Slowmist](https://slowmist.medium.com/the-root-cause-of-poly-network-being-hacked-ec2ee1b0c68f) |
| Direct Loss  | $610 million                                                                                     |
| Project Repo | [https://github.com/polynetwork/eth-contracts](https://github.com/polynetwork/eth-contracts)     |

## Contract Structure

There are two main contracts related to this incident: `EthCrossChainData(ECCD)` and `EthCrossChainManager(ECCM)`.

Under normal circumstances, Keepers monitor cross-chain transactions from the source chain to the destination chain, then submit block headers, proofs and other data to `ECCD` on destination chains.

Then in `ECCM`, the validity of cross-chain transactions will be checked and perform **contract** **calls** **specified by transactions** if all checks pass(eg. signed by Keepers, Merkle root correctness, etc.)**.** Those **contract** **calls** should be executing cross-chain operations as expected, but there were very weak restrictions.

### EthCrossChainData(ECCD)

* Update and store data of all cross-chain transactions.
* Add, change and store public keys of Keepers, who have the ability to move funds or perform other critical operations(eg. sign and submit cross-chain transactions witnessed from other chains).

**putCurEpochConPubKeyBytes()**

```
    function putCurEpochConPubKeyBytes(
        bytes memory curEpochPkBytes)
        public whenNotPaused onlyOwner returns (bool) {
            ConKeepersPkBytes = curEpochPkBytes;
            return true;
    }
```

Change public keys of Keepers. Because it has `onlyOwner` modifier, it can only be called by `ECCM`, the owner of `ECCD`.

### **EthCrossChainManager(ECCM)**

* Verify Poly chain header and proof, execute the cross-chain tx from Poly chain to Ethereum.
* Owner of the `ECCD` contract.

#### **changeBookKeeper()**

```
function changeBookKeeper(
        bytes memory rawHeader,
        bytes memory pubKeyList,
        bytes memory sigList)
    whenNotPaused public returns(bool) {
```

The ordinary and legal method for changing Keepers. Hacker was not able to call this one as it required Keepers' signatures.

**crossChain()**

```
function crossChain(
        uint64 toChainId,
        bytes calldata toContract,
        bytes calldata method,
        bytes calldata txData)
     whenNotPaused external returns (bool) {
```

This function was used for submitting a cross-chain tx on a source chain. Its info will be relayed to the `ECCD` contract on the destination chain by Keepers.

**verifyHeaderAndExecuteTx()**

```
function verifyHeaderAndExecuteTx(
        bytes memory proof,
        bytes memory rawHeader,
        bytes memory headerProof,
        bytes memory curRawHeader,
        bytes memory headerSig) {
```

Verify Poly chain header, proof, and signatures of Keepers and execute the cross-chain tx from Poly chain to Ethereum.

**\_executeCrossChainTx()**

```
function _executeCrossChainTx(
        address _toContract,
        bytes memory _method,
        bytes memory _args,
        bytes memory _fromContractAddr,
        uint64 _fromChainId)
    internal returns (bool){
```

An internal function called by **verifyHeaderAndExecuteTx()** to execute speificied operations.

## Attack Vectors & Details

### Holistic View

1. The hacker constructed sophisticated parameters for Keeper replacement.
2. The hacker called `crossChain()` from other chains to submit cross-chain transactions, which are not normal transactions but for Keeper replacement.
3. Relayers of Poly Network relayed those transactions to `ECCD` on the destination chain.
4. `ECCM` verified and executed malicious cross-chain transactions from `ECCD`.
5. Move funds as he wanted. Attack completed.

Except for Step1, the remainings are very ordinary operations, thus here we only delve into the details of Step1, malicious parameters.

### Construction of Malicious Parameters

Let's take a look at how does `_executeCrossChainTx()` execute calls to other contracts:

```
    (success, returnData) = _toContract.call(
        abi.encodePacked(
            bytes4(keccak256(abi.encodePacked(_method, "(bytes,bytes,uint64)"))),
            abi.encode(_args, _fromContractAddr, _fromChainId)));
```

Basically it can be interpreted as `ContractAddress.call(functionSelector, paramters).`

**Hash collision of function selector**

`functionSelector` = `bytes4(keccak256(abi.encodePacked(_method, "(bytes,bytes,uint64)")))`. With this `bytes4` typecasting, only the first 4 bytes were saved and Solidity function selector are 4 bytes long.

But here's how `abi.encodePacked()` works from Solidity Docs:

> If you use `keccak256(abi.encodePacked(a, b))` and both `a` and `b` are dynamic types, it is easy to craft collisions in the hash value by moving parts of `a` into `b` and vice-versa. More specifically, `abi.encodePacked("a", "bc") == abi.encodePacked("ab", "c")`. If you use `abi.encodePacked` for signatures, authentication or data integrity, make sure to always use the same types and check that at most one of them is dynamic. Unless there is a compelling reason, `abi.encode` should be preferred.

Namely, in `bytes4(keccak256(abi.encodePacked(_method, "(bytes,bytes,uint64)")))`, `"(bytes,bytes,uint64)"` is a very weak limitation, since `encodePacked` only concatenates all characters, and only the first four bytes of keccak256 hash were used as a function selector, the attacker can easily brute force an identical one.

The signature of `putCurEpochConPubKeyBytes(bytes)` , which updates the public keys of Keepers, is `0x41973cd9`. You can calculate it by `ethers.utils.id ('putCurEpochConPubKeyBytes(bytes)').slice(0, 10)` or various online tools.

The attacker brute-forced `RANDOM_STRING` to match the following equation:

`'0x41973cd9' == ethers.utils.id ('RANDOM_STRING(bytes,bytes,uint64)').slice(0, 10)`.

There are many `RANDOM_STRING` that meet the requirement above, eg. :

* `f1121318093`
* `func10487987874260605968`

**Omitted arguments**

There are three paramters in the codesnipet:

`abi.encode(_args, _fromContractAddr, _fromChainId)`

But there's only ONE parameter in the `putCurEpochConPubKeyBytes(bytes)` function. Is that a valid call if we call a function with redundant arguments? Conditionally, yes, they will just be omitted if you make up calls this way.

The hacker passed his address `0xA87fB85A93Ca072Cd4e5F0D4f178Bc831Df8a00B` to `_args`, which will be passed to `putCurEpochConPubKeyBytes(bytes memory curEpochPkBytes)` to replace Keepers address.

Construction of parameters done.

## Summary

* **Permission control matters**: In a complex project, `onlyOwner` or other forms of permission controls could fail. There could be other attack vectors to access the core. Developers should check with a more holistic picture to enforce solid permission controls.
* **Beware hash collision**: The ability to call arbitary or limited functions is good for expansible smart contract design. But using unsafe implementation could lead to hash collsion attack. It is suggested that developers change`call(bytes4(keccak256("f(uint256)")), a, b)` to`call(abi.encodeWithSignature("f(uint256)", a, b))`, and avoid `abi.encodePacked()`.

## Aftermath

Report from [Kudelski](https://research.kudelskisecurity.com)

> Poly Network asked the hacker to return the funds. The security company Slowmist published findings on the alleged hacker, claiming that the hacker’s identity had been exposed and that the group had access to the hacker’s email and IP address. According to Slowmist, the hacker was able to take advantage of a relatively unknown crypto exchange in Asia and they claimed to have a lot of information about the attacker.
>
> Whether this is true or not, the hacker started returning funds to Poly on Wednesday. By August 11th 15:00 UTC nearly half worth of tokens have been returned, and the hacker claims to be ready to return more in exchange for the unfreeze of the Tether tokens. A second message embedded in a transaction reads: **“IT’S ALREADY A LEGEND TO WIN SO MUCH FORTUNE. IT WILL BE AN ETERNAL LEGEND TO SAVE THE WORLD. I MADE THE DECISION, NO MORE DAO”**.
>
> While this story develops, it is not superfluous to remind that “blockchain” is not synonymous with “security”. It is very important to audit the security of your applications, including smart contracts.

## References

[https://docs.soliditylang.org/en/v0.8.15/abi-spec.html](https://docs.soliditylang.org/en/v0.8.15/abi-spec.html)

[https://research.kudelskisecurity.com/2021/08/12/the-poly-network-hack-explained/](https://research.kudelskisecurity.com/2021/08/12/the-poly-network-hack-explained/)

[https://slowmist.medium.com/the-root-cause-of-poly-network-being-hacked-ec2ee1b0c68f](https://slowmist.medium.com/the-root-cause-of-poly-network-being-hacked-ec2ee1b0c68f)
