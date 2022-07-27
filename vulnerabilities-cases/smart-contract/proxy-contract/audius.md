---
cover: >-
  https://images.unsplash.com/photo-1470225620780-dba8ba36b745?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw3fHxkanxlbnwwfHx8fDE2NTg4OTQ5MzA&ixlib=rb-1.2.1&q=80
coverY: -149.17985611510792
---

# Audius

## 摘要

Audius是一个去中心化的音乐平台。由于开发者未正确地使用代理合约，黑客对其多个合约进行了多次重初始化，来篡改项目的关键参数。

| 状态   | 已攻击                                                                  |
| ---- | -------------------------------------------------------------------- |
| 类型   | 代理                                                                   |
| 日期   | Jul 24, 2021                                                         |
| 来源   | [@BenWAGMI](https://twitter.com/BenWAGMI/status/1551163358434709505) |
| 直接损失 | $6M                                                                  |
| 项目仓库 | [https://github.com/AudiusProject](https://github.com/AudiusProject) |

## 有关账户

**Governance Contract** [https://etherscan.io/address/0x4deca517d6817b6510798b7328f2314d3003abac…](https://t.co/Ene6G5oeSX) (Proxy) [https://etherscan.io/address/0x35dd16dfa4ea1522c29ddd087e8f076cad0ae5e8…](https://t.co/IdSuGvAQhi) (Impl)

**Staking Contract** [https://etherscan.io/address/0xe6d97b2099f142513be7a2a068be040656ae4591…](https://t.co/lS0uCsIMqe) (Proxy) [https://etherscan.io/address/0xea10fd3536fce6a5d40d55c790b96df33b26702f…](https://t.co/npMyDZqWog) (Impl)

**DelegateManagerV2 Contract** [https://etherscan.io/address/0xf24aeab628493f82742db68596b532ab8a141057…](https://t.co/dkIaiFwyNh)

**Hacker’s EOA**

[https://etherscan.io/address/0xa0c7bd318d69424603cbf91e9969870f21b8ab4c…](https://t.co/hAkb2vlfNv)

**One of hacker's helper contracts**

[https://etherscan.io/address/0xbdbb5945f252bc3466a319cdcc3ee8056bf2e569](https://etherscan.io/address/0xbdbb5945f252bc3466a319cdcc3ee8056bf2e569)

## 攻击向量和袭击

### 整体布局

1. 通过重初始化篡改投票参数
2. 提交恶意提案
3. 通过重初始化篡改自己的投票比重
4. 投票
5. 执行提案

### 细节

#### 通过重初始化篡改投票参数

调用**Governance Contract**的`initilize()`来篡改投票参数：

设置 `votePeriod = 3` 和 `Delay = 0`, 使得提案被确认只需要3个区块的时间. 设置 `_votingQuorumPercent = 1%`, 意味着只需要有总质押数的1%即可通过提案。

#### 提交恶意提案

向Governance提交恶意提案（编号为85） 调用`submitProposal()`，其中 `_functionSignature = transfer(address,uint256)`，address是攻击者，数量为18,564,497,819,999,999,999,735,541，`_targetContractRegistryKey= 307800..00`（在registry合约中对应项目token地址）。提案成功将执行token的transfer。

#### 通过重初始化篡改自己的投票比重

`_quorumMet()`方法会检查投票是否达到额定人数，如果不到则无法出结果，不论是批准还是否决。黑客需要增加自己的权重来通过该检查。

```
    function _quorumMet(Proposal memory proposal, Staking stakingContract)
        internal view returns (bool)
        {
            uint256 participation = (
                (proposal.voteMagnitudeYes + proposal.voteMagnitudeNo)
                .mul(100)
                .div(stakingContract.totalStakedAt(proposal.submissionBlockNumber))
            );
            return participation >= votingQuorumPercent;
        }
```

在**DelegateManagerV2**中先调用初始化，将自己设置为`governanceAddress`，该地址有权限进行**delegatestake**。调用`delegatestake()`，可以看到该函数对`_amount`没有检查，是随意输入的数字。黑客输入了特别大的数字。这样他只需要投票yes就可以达到额定人数。

```
function delegateStake(
        address _targetSP,
        uint256 _amount
    ) external returns (uint256)
    {
        _requireIsInitialized();
        _requireStakingAddressIsSet();
        _requireServiceProviderFactoryAddressIsSet();
        _requireClaimsManagerAddressIsSet();

        require(
            !_claimPending(_targetSP),
            "DelegateManager: Delegation not permitted for SP pending claim"
        );
        address delegator = msg.sender;
        Staking stakingContract = Staking(stakingAddress);

        // Stake on behalf of target service provider
        stakingContract.delegateStakeFor(
            _targetSP,
            delegator,
            _amount
        );

        // Update list of delegators to SP if necessary
        if (!_delegatorExistsForSP(delegator, _targetSP)) {
            // If not found, update list of delegates
            spDelegateInfo[_targetSP].delegators.push(delegator);
            require(
                spDelegateInfo[_targetSP].delegators.length <= maxDelegators,
                "DelegateManager: Maximum delegators exceeded"
            );
        }

        // Update following values in storage through helper
        // totalServiceProviderDelegatedStake = current sp total + new amount,
        // totalStakedForSpFromDelegator = current delegator total for sp + new amount,
        // totalDelegatorStake = current delegator total + new amount
        _updateDelegatorStake(
            delegator,
            _targetSP,
            spDelegateInfo[_targetSP].totalDelegatedStake.add(_amount),
            delegateInfo[delegator][_targetSP].add(_amount),
            delegatorTotalStake[delegator].add(_amount)
        );

        require(
            delegateInfo[delegator][_targetSP] >= minDelegationAmount,
            ERROR_MINIMUM_DELEGATION
        );

        // Validate balance
        ServiceProviderFactory(
            serviceProviderFactoryAddress
        ).validateAccountStakeBalance(_targetSP);

        emit IncreaseDelegatedStake(
            delegator,
            _targetSP,
            _amount
        );

        // Return new total
        return delegateInfo[delegator][_targetSP];
    }
    
function delegateStakeFor(
        address _accountAddress,
        address _delegatorAddress,
        uint256 _amount
    ) external {
        _requireIsInitialized();
        _requireDelegateManagerAddressIsSet();

        require(
            msg.sender == delegateManagerAddress,
            ERROR_ONLY_DELEGATE_MANAGER
        );
        _stakeFor(
            _accountAddress,
            _delegatorAddress,
            _amount);
    }
```

#### 为恶意提案投票

黑客调用**Governance**中的`submitVote()`为85号提案投赞成票，此时区块编号为15201796。

#### 执行恶意提案

黑客调用**Governance**中的`evaluateProposalOutcome()`方法为85号提案结算，此时区块高度为15201799，已经过了他设置的3个区块的投票窗口，也达到了额定人数，也只有他进行了投票（赞成票），提案自然通过，并自动执行提案中提交的方法，也即向黑客打币。

## 重初始化

黑客能够重初始化一个已初始化的合约是因为不正确地使用了代理合约架构。

### 代理合约架构

代理架构可以简单分为代理(Proxy)和实现(Impl)两个合约。实际上不止两个，有其他的辅助性合约。

![Simplified Proxy Architecture](<../../../.gitbook/assets/Proxy Architecture.png>)

用户与合约交互时，Proxy合约中并没不“知道”Impl中有什么方法和变量。想调用Impl中的方法实际上是从Proxy中通过**delegatecall**调用的。

**Delegatecall的主要特性**: 将被调用者的代码拿到自己的内部运行。也就是说，最终运行的结果，比如状态变量等，**全部存储在Proxy里**。这也是代理合约可升级的基础。而这些变量并不按名称索引，而是存储槽查找，这样就会有一个问题，即存储冲突。

|        | Proxy        | Impl            | Note       |
| ------ | ------------ | --------------- | ---------- |
| Slot 0 | address impl | <- bool var1    | Collision! |
| Slot 1 |              | <- bool var2    |            |
| Slot 2 |              | <- uint256 var3 |            |
| Slot 3 |              |                 |            |
|        |              |                 |            |
|        |              |                 |            |

Proxy中是要用一个变量记录Impl合约的地址的，上表中已经冲突了。为解决该问题，EIP-1967定义了一个存储槽的位置`keccak256('eip1967.proxy.implementation')) - 1`，并将impl地址存储在该槽内。即表中的`IMPLEMENTATION_SLOT`。这是个constant，定义它并不需要占用存储槽。然后用solidity汇编将变量写入指定的槽。

```
  bytes32 internal constant IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;

  /**
   * @dev Returns the current implementation.
   * @return Address of the current implementation
   */
  function _implementation() internal view returns (address impl) {
    bytes32 slot = IMPLEMENTATION_SLOT;
    assembly {
      impl := sload(slot)
    }
  }
```

由此解决了储存冲突。

|             | Proxy        | Impl            | Note         |
| ----------- | ------------ | --------------- | ------------ |
| Slot 0      |              | <- bool var1    |              |
| Slot 1      |              | <- bool var2    |              |
| Slot 2      |              | <- uint256 var3 |              |
| Slot 3      |              |                 |              |
| ...         |              |                 |              |
| Slot Custom | address impl |                 | No collison! |
|             |              |                 |              |

### Audius中的情况

There were a customized variable `proxyAdmin` in its **multiple proxy contracts**, which caused the `Initalized` and `Initializing` variables in Impl that marked the initialization state to conflict with the `proxyAdmin` storage.

Audius在**多个Proxy合约**里自定义了一个`proxyAdmin`变量存储管理员地址，导致了Impl中两个标记初始化状态的`Initalized`和`Initializing`变量与`proxyAdmin`的存储有所冲突，

```
contract AudiusAdminUpgradeabilityProxy is UpgradeabilityProxy {
    address private proxyAdmin;
    string private constant ERROR_ONLY_ADMIN = (
        "AudiusAdminUpgradeabilityProxy: Caller must be current proxy admin"
    );

    /**
     * @notice Sets admin address for future upgrades
     * @param _logic - address of underlying logic contract.
     *      Passed to UpgradeabilityProxy constructor.
     * @param _proxyAdmin - address of proxy admin
     *      Set to governance contract address for all non-governance contracts
     *      Governance is deployed and upgraded to have own address as admin
     * @param _data - data of function to be called on logic contract.
     *      Passed to UpgradeabilityProxy constructor.
     */
    constructor(
      address _logic,
      address _proxyAdmin,
      bytes memory _data
    )
    UpgradeabilityProxy(_logic, _data) public payable
    {
        proxyAdmin = _proxyAdmin;
    }
    
    //...
    
}
```

我们在Rinkeby上进行了测试发现，`initialized`与`initializing`在一开始就都处于出错状态(等于True),因为`address`类型可以以非0值覆盖多个`bool`类型(uint 8)。

由于`initializing == true`, 修饰符总认为可以进行初始化。

```
modifier initializer() {
    bool isTopLevelCall = !_initializing;
    require(
        (isTopLevelCall && _initialized < 1) || (!Address.isContract(address(this)) && _initialized == 1),
        "Initializable: contract is already initialized"
    );
    _initialized = 1;
    if (isTopLevelCall) {
        _initializing = true;
    }
    _;
    if (isTopLevelCall) {
        _initializing = false;
        emit Initialized(1);
    }
}
```

在测试中，如果移除`proxyAdmin`参数，则无法重初始化。所以结论是很明确的。

## 总结

该攻击能够发生是由于对代理合约结构的错误理解。

开发者在使用OpenZeppelin代理合约前应该熟读[OpenZeppelin Proxy Docs](https://docs.openzeppelin.com/contracts/4.x/api/proxy)。任何自定义都应该小心谨慎。进行权限管理应该使用标准方法。如果出于特殊目的需要自定义，应该使用EIP1967中的存储方法。

## 参考

[https://docs.openzeppelin.com/contracts/4.x/api/proxy](https://docs.openzeppelin.com/contracts/4.x/api/proxy)

[https://eips.ethereum.org/EIPS/eip-1967](https://eips.ethereum.org/EIPS/eip-1967)
