# Sleep Minting

## Abstract

A scam technique makes NFT mint & transfer look like originated from a famous account, and lures users to buy the scam NFTs in the collection.

## Mechanism

Most dApps or websites are using Event logs of Mint & Transfer solely when analysing its origin, sender, receiver and etc..

`event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);`

The front end just listens to this event emitted from blockchain, and displays to users that some NFT was transferred from/to some addresses according to this event log.



The following is how `transferFrom()` works:

1. Check if the caller is the owner of the NFT to be transferred
2. In the end, it will emit a `Transfer` event.

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



Now we add a private address in the NFT721 contract,

`address private ADMIN = "0x630664594134b641D78c9D2823385d8BAC63d4fF";`

Then we modifiy the requirement in `transferFrom()` and grant a super power to the ADMIN.

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

ADMIN has the ability to transfer anyone's NFT while the `Transfer` Event remains the same, who still report the ordinary from/to address. Thus, the front end will also resolve the result as usual, but this time it's wrong.

## Countermeasure

For front end, it's easy to filter a Sleep Minting action:

```
// PESUDO CODE
if (transferEvent.from != tx.origin && Approval[_tokenId] != tx.origin){
    "Sleep minted";
}
```

Though this may have side effects. You should modify the condition according to your use case.

## References

[https://a16z.com/2022/03/09/sleep-minting-nfts/](https://a16z.com/2022/03/09/sleep-minting-nfts/)
