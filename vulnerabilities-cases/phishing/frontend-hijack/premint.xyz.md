# Premint.xyz

## 摘要

PREMINT是一个有多种功能的NFT工具网站。其网站前端被黑客劫持后注入了包含恶意`SetApprovalForAll`函数的JavaScript代码。

| 状态       | 已修复                                       |
| ------------ | ------------------------------------------- |
| 类型         | 钓鱼                                    |
| 日期         | July 17, 2022                               |
| 来源       | [PREMINT](https://twitter.com/PREMINT\_NFT) |
| 直接损失  | \~$400K                                     |
| 项目仓库 | -                                           |

## 攻击路径

### **SetApproveForAll**

```
    mapping(address => mapping(address => bool)) private _operatorApprovals;

    function setApprovalForAll(address operator, bool approved) public virtual override {
        _setApprovalForAll(_msgSender(), operator, approved);
    }

    function _setApprovalForAll(
        address owner,
        address operator,
        bool approved
    ) internal virtual {
        require(owner != operator, "ERC721: approve to caller");
        _operatorApprovals[owner][operator] = approved;
        emit ApprovalForAll(owner, operator, approved);
    }
```
当发送NFT时，ERC721合约会检查条件 `spender == owner || isApprovedForAll(owner, spender) || getApproved(tokenId) == spender.`

`isApprovedForAll` 会检查 `_operatorApprovals` 这个mapping是true还是false。

`setApprovalForAll` 可以将 `operator` 地址设置为特定owner在该NFT collection中的高权限账户。

### 前端劫持

攻击者向premint.xyz注入了恶意的JS代码。

恶意代码通过URL https://s3-redwood-labs-premint-xyz\[.]com/cdn.min.js?v=1658046560357 注入了网站，不过由于DNS已不存在，该文件现已无法访问。

所有当时与该网站有交互的用户都容易遭受攻击。

## 反制措施

### 开发者

开发者应加强服务端安全避免类似攻击。

如果可能，应使用永久性文件存储设施如IPFS，并配合不同的URL进行版本控制。

### 用户

用户应该对任何有关`Approve`的交易保持谨慎。如果不明白为什么网站请求该许可，最好拒绝。

也可以使用一些针对Metamask或类似钱包的反钓鱼、反恶意交易插件。

## 参考

[https://www.certik.com/resources/blog/77oaazrsx1mewnraJePYQI-premint-nft-incident-analysis](https://www.certik.com/resources/blog/77oaazrsx1mewnraJePYQI-premint-nft-incident-analysis)
