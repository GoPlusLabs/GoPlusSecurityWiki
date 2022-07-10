---
cover: ../../.gitbook/assets/xafAmLjc.jpeg
coverY: 0
---

# ERC721R Bug

## 摘要

| 状态       | 已修复                                                                    |
| ------------ | ------------------------------------------------------------------------ |
| 类型         | 合约                                                                 |
| 日期         | Apr 12, 2022                                                             |
| 来源       | [@BenWAGMI](https://twitter.com/BenWAGMI/status/1513793556367884289)     |
| 直接损失  | 无，早期阶段项目                                |
| 项目仓库 | [https://github.com/erc721r/ERC721R](https://github.com/erc721r/ERC721R) |

## NFT721R是什么?

一个允许NFT铸造者在一定时间内退款的NFT协议。

## 问题

正常情况下，开发者在`refundEndTime`后调用`withdraw()`函数来取出铸造过程中收集的费用，这一步骤没有问题。

```
function withdraw() external onlyOwner {
    require(block.timestamp > refundEntime), "Refund period not over");
    uint256 balance = address(this).balance;
    Address.sendValue(payable(owner)), blance);
}
```

再来看一下refund()函数： 铸造者调用该函数将铸造的NFT退回至 `refundAddress`(一个由开发者控制和设置的地址) 然后再从合约拿回相应的数量的ETH。可如果`refundAddress`本身就是一个铸造者呢？

```
function refund(uint256 [] calldata tokenIds) external {
    require(refundGuaranteeActive(), "Refund expired");

    for (uint256 i = 0; i < tokenIds.length; i++) {
        uint256 tokenId = tokenIds[i];
        require(msg.sender == ownerOf(tokenId), "Not token owner");
        transferFrom(msg.sender, refundAddress, tokenId);
    }

    uint256 refundAmount = tokenIds.length * mintPrice;
    Address.sendValue(payable(msg.sender)), refundAmount);
}
```

恶意开发者可以设置一个`refundAddress`，然后再从该地址铸造NFT。下一步，调用`refund()`。由于NFT总会退回`refundAddress`，他仍然持有这个NFT同时又得到了一些ETH。他可以重复该过程多次直至偷走所有资金。
