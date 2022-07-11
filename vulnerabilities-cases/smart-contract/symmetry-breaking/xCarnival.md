---
cover: ../../../.gitbook/assets/Xcarnival.jpg
coverY: 0
---

# XCarnival

## 摘要

NFT借贷平台[@XCarnival\_Lab](https://twitter.com/XCarnival\_Lab)遭到了攻击，至少价值$3.8M的以太币被盗。该平台有个Bug：在取出抵押的NFT后，其orderID仍然可用于贷款。

| 状态       | 被攻击                                                            |   |
| ------------ | -------------------------------------------------------------------- | - |
| 类型         | 合约，对称性破缺                                          |   |
| 日期         | Jun 26, 2022                                                         |   |
| 来源       | [@BenWAGMI](https://twitter.com/BenWAGMI/status/1541145543514411008) |   |
| 直接损失  | $3.8M                                                                |   |
| 项目仓库 | [https://github.com/xcarnival](https://github.com/xcarnival)         |   |

## 合约结构

### [xETH](https://etherscan.io/token/0xb38707e31c813f832ef71c70731ed80b45b85b2d)

* **xToken**的一个实例，用来存放资金的合约。资金也从这里借出。
* **borrow()**，当用户请求贷款时会调用该方法

### [xNFT](https://etherscan.io/address/0xb14b3b9682990ccc16f52eb04146c3ceab01169a#code)

* NFT抵押、取回等操作的管理者
* **pledgeAndBorrow()** 负责抵押NFT并从xToken借款
* **withdrawNFT()**  用来取回NFT

### [P2Controller](https://etherscan.io/address/0xb7e2300e77d81336307e36ce68d6909e43f4d38a)

* 很多借贷限制的检查者
* **borrowAllowed()** 验证一个orderId是否有效

## 攻击向量与详情

### 整体布局

1. 调用`pledgeAndBorrow()`，将NFT质押金xETH中，但什么都不借贷（amount=0)，此过程会生成一个`orderID`
2. 调用`withdrawNFT()`取出NFT。该过程中，合约并没有取消对应的`orderID`
3. 用`orderID`进行贷款

### 细节

#### 准备工作&#x20;

[Hacker](https://etherscan.io/address/0xb7cbb4d43f1e08327a90b32a8417688c9d0b800a)从Tornado获得启动资金。然后从OpenSea购买了[#BAYC](https://twitter.com/hashtag/BAYC?src=hashtag\_click) 5110。



**部署总控合约**

黑客部署了[总控合约](https://etherscan.io/address/0xf70f691d30ce23786cfb3a1522cfd76d159aca8d), 该总控合约生成了许多个用来当女巫进行借款的马仔, 如 [Slave 5338](https://etherscan.io/address/0x53386a82e55202a74c6d83c7eede7a80ba553714).



#### 创建多个orderIDs

**总控**将BAYC 5110发送给**马仔**(eg, 0x5338…)。马仔接着调用 `xNFT`中的`pledgeAndBorrow()`,用BAYC抵押并什么也不借(传入虚假的xToken以及0 amount).&#x20;

该步骤生成了orderID(43)生成。

然后**马仔533**取回了该NFT并发回至**总控**，总控再与其他马仔重复该过程。通过这种形式创建了许多orderID，之后可以用来作为贷款凭证，因为有问题的**`xNFT`**合约并没有撤销该orderID的效力：

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



#### 借款

那么下一步就是，**总控**依次调用所有的**马仔**，从`xETH`合约中借钱。攻击完成。NFT早就被取走，黑客从虚空中借到了大笔资金。&#x20;

其中一笔交易：

[https://t.co/gyYFyTt8wy](https://t.co/gyYFyTt8wy)


在`xETH`中，`borrow()`会调用`borrowInternal()`然后是`controller.borrowAllowed()`来验证orderID是否有效。

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

这个是P2controller中的`borrowAllowed()`。首先它会询问`xNFT.getOrderDetail()`。这其中有许多限制，但没有一个能阻止黑客。注意：黑客需要多个马仔是因为下面有一个可借贷数量检查。

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

## 总结

问题在于在抵押品取走后仍可以借款。&#x20;

开发者在成对的行为中应注意安全的对称性。

## 参考

[https://twitter.com/BenWAGMI/status/1541145543514411008](https://twitter.com/BenWAGMI/status/1541145543514411008)
