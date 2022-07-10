---
cover: ../../../.gitbook/assets/Xcarnival.jpg
coverY: 0
---

# XCarnival

## Abstract

NFT lending platform[@XCarnival\_Lab](https://twitter.com/XCarnival\_Lab) was exploited. At least 3000 [$ETH](https://twitter.com/search?q=%24ETH\&src=cashtag\_click)(\~$3.8M) was stolen. There was a bug in the NFT platform: After you withdraw your collateralised NFT, its orderID is still there available for loan request.

| Status       | Exploited                                                            |   |
| ------------ | -------------------------------------------------------------------- | - |
| Type         | Contract, Symmtery Breaking                                          |   |
| Date         | Jun 26, 2022                                                         |   |
| Source       | [@BenWAGMI](https://twitter.com/BenWAGMI/status/1541145543514411008) |   |
| Direct Loss  | $3.8M                                                                |   |
| Project Repo | [https://github.com/xcarnival](https://github.com/xcarnival)         |   |

## Contract Structure

### [xETH](https://etherscan.io/token/0xb38707e31c813f832ef71c70731ed80b45b85b2d)

* An instance of **xToken**, a contract for holding funds. Funds is borrowed from here
* **borrow()** will be called when users request a loan

### ****[xNFT](https://etherscan.io/address/0xb14b3b9682990ccc16f52eb04146c3ceab01169a#code)

* Manager of NFT collateralisation, withdrawing, etc..
* **pledgeAndBorrow()** is in charge of depositing NFT as collateral and borrowing from xToken
* **withdrawNFT()**  for NFT withdraw

### [P2Controller](https://etherscan.io/address/0xb7e2300e77d81336307e36ce68d6909e43f4d38a)

* the checker for many lending restrictions
* **borrowAllowed()** verifies if an orderID is valid.

## Attack Vectors & Details

### Holistic View

1. Pledge an NFT into xETH and borrow nothing(amount = 0), an `orderID` will be generated after `pledgeAndBorrow()`
2. `withdrawNFT()` to take the NFT back. In this step, the contract won't nullify the `orderID`
3. Request loan by `orderID`

### Details

#### Preparation&#x20;

[Hacker](https://etherscan.io/address/0xb7cbb4d43f1e08327a90b32a8417688c9d0b800a) funded his account from Tornado. Then bought [#BAYC](https://twitter.com/hashtag/BAYC?src=hashtag\_click) 5110 from OpenSea.



**Deploy the Master contract**

He deployed a [Master contract](https://etherscan.io/address/0xf70f691d30ce23786cfb3a1522cfd76d159aca8d), which derived many Slave contracts as sybils to use the same NFT for borrowing, eg. [Slave 5338](https://etherscan.io/address/0x53386a82e55202a74c6d83c7eede7a80ba553714).



#### Create many orderIDs

**Master** transferred BAYC 5110 to **Slave**(eg, 0x5338…). Slave then called `pledgeAndBorrow()` function in `xNFT`, with the BAYC and borrowed nothing(with a fake xToken and 0 amount).&#x20;

In this step an orderID (43) was generated.



Then **Slave 5338** withdrew the NFT and sent it back to **Master**, who then repeated this process with other Slaves. In this way they created many orderIDs, which can later be used as lending credentials since bugged **`xNFT`** contract didn’t revoke the credential after withdrawing:

```
    function withdrawNFT(uint256 orderId) external nonReentrant whenNotPaused(2){
        LiquidatedOrder storage liquidatedOrder = allLiquidatedOrder[orderId];
        Order storage _order = allOrders[orderId];
        if(isOrderLiquidated(orderId)){
            //...
        }else{
            require(!_order.isWithdraw, "the order has been drawn");
            require(_order.pledger != address(0) && msg.sender == _order.pledger, "withdraw auth failed");
            uint256 borrowBalance = controller.getOrderBorrowBalanceCurrent(orderId);
            require(borrowBalance == 0, "order has debt");
            transferNftInternal(address(this), _order.pledger, _order.collection, _order.tokenId, _order.nftType);
        }
        _order.isWithdraw = true;
        emit WithDraw(_order.collection, _order.tokenId, orderId, _order.pledger, msg.sender);
    }

```



#### Borrow

So next step the **Master** called all **Slaves**, in turn, to borrow $ETH from `xETH` contract. Attack completed. The hacker borrowed money from void(collateral NFT had already been withdrawn).&#x20;

One of the tx:

[https://t.co/gyYFyTt8wy](https://t.co/gyYFyTt8wy)



In `xETH`, `borrow()`will call `borrowInternal()` then `controller.borrowAllowed()` to verify if an orderID is valid.

borrow()

```
function borrow(uint256 orderId, address payable borrower, uint256 borrowAmount) external{
    require(msg.sender == borrower || tx.origin == borrower, "borrower is wrong");
    accrueInterest();
    borrowInternal(orderId, borrower, borrowAmount);
}
```

borrowInternal()

```
function borrowInternal(uint256 orderId, address payable borrower, uint256 borrowAmount) internal nonReentrant{
    
    controller.borrowAllowed(address(this), orderId, borrower, borrowAmount);

    require(accrualBlockNumber == getBlockNumber(),"block number check fails");
    
    require(getCashPrior() >= borrowAmount, "insufficient balance of underlying asset");

    BorrowLocalVars memory vars;

    vars.orderBorrows = borrowBalanceStoredInternal(orderId); //first time:0
    vars.orderBorrowsNew = addExp(vars.orderBorrows, borrowAmount); // first time: 0
    vars.totalBorrowsNew = addExp(totalBorrows, borrowAmount); //first time: total in pool
    
    doTransferOut(borrower, borrowAmount);

    orderBorrows[orderId].principal = vars.orderBorrowsNew;
    orderBorrows[orderId].interestIndex = borrowIndex;

    totalBorrows = vars.totalBorrowsNew;

    controller.borrowVerify(orderId, address(this), borrower);

    emit Borrow(orderId, borrower, borrowAmount, vars.orderBorrowsNew, vars.totalBorrowsNew);
}

```

Here is the `borrowAllowed()` in P2controller. It will first ask `xNFT.getOrderDetail().` There are many other restrictions, but none of them can stop the hacker. Note: the reason the hacker needed multiple slaves is there is an amount checker for a single order at the bottom.

```
function borrowAllowed(address xToken, uint256 orderId, address borrower, uint256 borrowAmount) external whenNotPaused(xToken, 3){
    require(poolStates[xToken].isListed, "token not listed"); // called from xETH, TRUE

    orderAllowed(orderId, borrower);

    (address _collection , , ) = xNFT.getOrderDetail(orderId);

    CollateralState storage _collateralState = collateralStates[_collection];
    require(_collateralState.isListed, "collection not exist"); // BAYC had been added to collateral list, TRUE
    require(_collateralState.supportPools[xToken] || _collateralState.isSupportAllPools, "collection don't support this pool"); // xETH, ofc TRUE

    address _lastXToken = orderDebtStates[orderId]; //    mapping(uint256 => address) public orderDebtStates;
    //  It will be 0 for the first time for an Order, but will be set to xETH in the end.
    require(_lastXToken == address(0) || _lastXToken == xToken, "only support borrowing of one xToken"); // easy, TRUE

    (uint256 _price, bool valid) = oracle.getPrice(_collection, IXToken(xToken).underlying());
    require(_price > 0 && valid, "price is not valid"); // oracle price feed, TRUE

    // Borrow cap of 0 corresponds to unlimited borrowing
    if (poolStates[xToken].borrowCap != 0) {
        require(IXToken(xToken).totalBorrows().add(borrowAmount) < poolStates[xToken].borrowCap, "pool borrow cap reached"); // if pool have enough funds to borrow, TRUE
    }

    uint256 _maxBorrow = mulScalarTruncate(_price, _collateralState.collateralFactor);
    uint256 _mayBorrowed = borrowAmount;
    if (_lastXToken != address(0)){
        _mayBorrowed = IXToken(_lastXToken).borrowBalanceStored(orderId).add(borrowAmount);  
    }
    require(_mayBorrowed <= _maxBorrow, "borrow amount exceed"); //amount check. TRUE

    if (_lastXToken == address(0)){
        orderDebtStates[orderId] = xToken;
    }
}

```

## Summary

Collateral is still valid after withdrawing.&#x20;

Developers should be aware of the security symmetry in paired actions.

## References

[https://twitter.com/BenWAGMI/status/1541145543514411008](https://twitter.com/BenWAGMI/status/1541145543514411008)
