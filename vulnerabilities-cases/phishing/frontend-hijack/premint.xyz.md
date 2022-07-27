# Premint.xyz

## Abstract

PREMINT is an NFT tool website with various convenient features. Their website frontend was hijacked and injected malicious JavaScript code with `SetApprovalForAll` function.

| Status       | Fixed                                       |
| ------------ | ------------------------------------------- |
| Type         | Phishing                                    |
| Date         | July 17, 2022                               |
| Source       | [PREMINT](https://twitter.com/PREMINT\_NFT) |
| Direct Loss  | \~$400K                                     |
| Project Repo | -                                           |

## Attack Vector

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

While transferring NFT, ERC721 contract will check condition `spender == owner || isApprovedForAll(owner, spender) || getApproved(tokenId) == spender.`

`isApprovedForAll` will check whether the mapping `_operatorApprovals` is true or false.

`setApprovalForAll` can change any address as `operator` for all NFTs of a given owner in one collection.

### Frontend Injection

A hacker uploaded malicious JavaScript code to premint.xyz, which compromised the website.&#x20;

The malicious code was injected into the website via URL: https://s3-redwood-labs-premint-xyz\[.]com/cdn.min.js?v=1658046560357, however the file is no longer available due to the Domain Name Server no longer existing.

All users who were interacting with their frontend were vulnerable to the attack.

## Countermeasures

### For Devs

Developers should enhance server-side safety to avoid similar attacks.

If possible, it's recommended to host the website in an immutable file system like IPFS and upgrade your website with versions in URL.

### For Users

Users should beware of anything related to `Approve`. If you don't understand why the website is asking for that permission, it's better to reject the request.

You can also install anti-phishing/malicious transaction extensions for Metamask and other similar wallets.

## References

[https://www.certik.com/resources/blog/77oaazrsx1mewnraJePYQI-premint-nft-incident-analysis](https://www.certik.com/resources/blog/77oaazrsx1mewnraJePYQI-premint-nft-incident-analysis)
