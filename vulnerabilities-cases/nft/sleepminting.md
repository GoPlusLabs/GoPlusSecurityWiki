---
cover: >-
  https://images.unsplash.com/photo-1492684223066-81342ee5ff30?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw0fHxldmVudHxlbnwwfHx8fDE2NTY5MDcyODQ&ixlib=rb-1.2.1&q=80
coverY: 0
---

# Sleep Minting

## 摘要

一种NFT诈骗手段，可以让NFT的mint和transfer看起来像来自某个著名账户，以此诱骗用户购买该 NFT集合中的物品。

## 机制

大部分dApp或网站都仅仅根据Mint和Transfer的事件日志来分析其交易过程，如源头，发送者，接受者等等。

`event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);`

前端仅监听这个从链上发出的事件，并据此告知用户该NFT发送自/至某些地址。

下面是`transferFrom()`函数如何工作的:

1. 检查调用者是否是该NFT的owner
2. 调用成功后会生成 `Transfer` 事件.

```
  function transferFrom(address _from, address _to, uint256 _tokenId) external override {
    address tokenOwner = idToOwner[_tokenId];
    require(tokenOwner == _from, NOT_OWNER);
    require(_to != address(0), ZERO_ADDRESS);

    _transfer(_to, _tokenId);
  }


  function _transfer(address _to, uint256 _tokenId) internal virtual {
    address from = idToOwner[_tokenId];
    _clearApproval(_tokenId);

    _removeNFToken(from, _tokenId);
    _addNFToken(_to, _tokenId);

    emit Transfer(from, _to, _tokenId);
  }
```

现在我们在NFT721合约中增加一个私有地址，

`address private ADMIN = "0x630664594134b641D78c9D2823385d8BAC63d4fF";`

然后再修改 `transferFrom()`中的条件限制，赋予该地址超级权限。

```
  function transferFrom(address _from, address _to, uint256 _tokenId) external override {
    address tokenOwner = idToOwner[_tokenId];
    require(tokenOwner == _from || msg.sender == ADMIN, NOT_OWNER);
    //...
    
    _transfer(_to, _tokenId);
  }


  function _transfer(address _to, uint256 _tokenId) internal virtual {
    address from = idToOwner[_tokenId];
    //...
    emit Transfer(from, _to, _tokenId);
  }
```

ADMIN有转移任何人NFT的权限，并且其`Transfer`事件还和之前一样，始终报告原有的from/to和地址。因此前端会照常解析，但就被钓鱼了。

## 反制措施

对前端而言，过滤Sleep Minting行为是比较简单的：

```
// PESUDO CODE
if (transferEvent.from != tx.origin && Approval[_tokenId] != tx.origin){
    "Sleep minted";
}
```

不过这可能有其他副作用，需要根据使用场景进行修改。

## 参考

[https://a16z.com/2022/03/09/sleep-minting-nfts/](https://a16z.com/2022/03/09/sleep-minting-nfts/)
