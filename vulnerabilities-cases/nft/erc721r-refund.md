---
cover: ../../.gitbook/assets/xafAmLjc.jpeg
coverY: 0
---

# ERC721R Bug

## Abstract

| Status       | Fixed                                                                    |   |
| ------------ | ------------------------------------------------------------------------ | - |
| Type         | Contract                                                                 |   |
| Date         | Apr 12, 2022                                                             |   |
| Source       | [@BenWAGMI](https://twitter.com/BenWAGMI/status/1513793556367884289)     |   |
| Direct Loss  | None. It was reported in the early stage.                                |   |
| Project Repo | [https://github.com/erc721r/ERC721R](https://github.com/erc721r/ERC721R) |   |

## What is NFT721R?

An NFT protocol enabling minters to return the minted NFT for a refund in a certain period.

## Issue

In the typical case, the NFT dev calls this `withdraw()` function after refundEndTime to withdraw the Eth raised from minting. This step is OK.

```
function withdraw() external onlyOwner {
    require(block.timestamp > refundEntime), "Refund period not over");
    uint256 balance = address(this).balance;
    Address.sendValue(payable(owner)), blance);
}
```

But, check the refund function: Minter calls this func to return the NFTs he minted to the `refundAddress`(an address set and controlled by dev) then gets the corresponding amount of [$ETH](https://twitter.com/search?q=%24ETH\&src=cashtag\_click) back from the NFT contract. But what if `refundAddress` is a minter holding one of the NFTs?

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

A scam dev will set a refundAddress, then mint an NFT with this `refundAddress`. Next step, he calls `refund()`. Because the NFT will always return to refundAddress, he still possesses that NFT while collecting some amount of Eth. He can do it multiple times until all funds run out.

