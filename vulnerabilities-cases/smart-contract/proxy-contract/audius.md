---
cover: >-
  https://images.unsplash.com/photo-1470225620780-dba8ba36b745?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw3fHxkanxlbnwwfHx8fDE2NTg4OTQ5MzA&ixlib=rb-1.2.1&q=80
coverY: -149.17985611510792
---

# Audius

## Abstract

Audius is a fully decentralized music platform. It was hacked due to improper proxy contract usage. The hacker re-initialised multiple contracts of the project to tamper with malicious parameters.

| Status       | Exploited                                                            |
| ------------ | -------------------------------------------------------------------- |
| Type         | Proxy                                                                |
| Date         | Jul 24, 2021                                                         |
| Source       | [@BenWAGMI](https://twitter.com/BenWAGMI/status/1551163358434709505) |
| Direct Loss  | $6M                                                                  |
| Project Repo | [https://github.com/AudiusProject](https://github.com/AudiusProject) |

## Related Accounts

**Governance Contract** [https://etherscan.io/address/0x4deca517d6817b6510798b7328f2314d3003abac…](https://t.co/Ene6G5oeSX) (Proxy) [https://etherscan.io/address/0x35dd16dfa4ea1522c29ddd087e8f076cad0ae5e8…](https://t.co/IdSuGvAQhi) (Impl)

**Staking Contract** [https://etherscan.io/address/0xe6d97b2099f142513be7a2a068be040656ae4591…](https://t.co/lS0uCsIMqe) (Proxy) [https://etherscan.io/address/0xea10fd3536fce6a5d40d55c790b96df33b26702f…](https://t.co/npMyDZqWog) (Impl)\
\
**DelegateManagerV2 Contract** [https://etherscan.io/address/0xf24aeab628493f82742db68596b532ab8a141057…](https://t.co/dkIaiFwyNh)&#x20;

**Hacker’s EOA**&#x20;

[https://etherscan.io/address/0xa0c7bd318d69424603cbf91e9969870f21b8ab4c…](https://t.co/hAkb2vlfNv)&#x20;

**One of hacker's helper contracts**

[https://etherscan.io/address/0xbdbb5945f252bc3466a319cdcc3ee8056bf2e569](https://etherscan.io/address/0xbdbb5945f252bc3466a319cdcc3ee8056bf2e569)



## Attack Vectors & Details

### Holistic View

1. Tamper with vote parameters by re-initialisation
2. Submit malicious proposal&#x20;
3. Tamper with vote weight by re-initialisation
4. Vote&#x20;
5. Execute proposal

### Details

#### Tamper with vote parameters by re-initialisation

Call `initilize()` of **Governance Contract** to tamper with vote params:

Set `votePeriod = 3` and `Delay = 0`, so that only three blocks were needed to finish voting. Set `_votingQuorumPercent = 1%`, meaning that only 1% of the total staked token will meet the quorum.

#### Submit malicious proposal&#x20;

Submit malicious proposal(id=85) to Governance: Call `submitProposal()` with `_functionSignature = transfer(address,uint256)`, `address = attacker`, `amount = 18,564,497,819,999,999,999,735,541`, `_targetContractRegistryKey= 307800..00`(resolve as token contract in registry).

#### Tamper with vote weight by re-initialisation

\_quorumMet() will check whether the vote has met the quorum. If not then no result will come out. The attacker needed to increase its own weight to pass this check.

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

In DelegateManagerV2 call `initilize()`, set himself as `governanceAddress`, who has the power to **delegatestake**. Call `delegatestake()`, as we can see there’s no limitation on \_amount, so the hacker wrote a very large number. Only vote yes to meet the quorum.

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

#### Vote

[Call submitVote()](https://etherscan.io/tx/0x3c09c6306b67737227edc24c663462d870e7c2bf39e9ab66877a980c900dd5d5) to vote for proposal 85. Block height was 15201796 at that time.

#### Execute proposal

Call `evaluateProposalOutcome()` in **Governance** to settle proposal 85. Block height was 15201799. Settlement time window reached, quorum met, and only him voted for yes, so the proposal passed. Function inside the proposal will be called, i.e. transfer token to the attacker.

## Re-initialisation

The reason why the hacker could re-initialise an initialised contract is improper usage of the proxy architecture.

### Proxy Architecture

A proxy architecture can be simply divided into two contracts: proxy(Proxy) and implementation (Impl). In fact, there are some other auxiliaries, more than two contracts.

![Simplified Proxy Architecture](<../../../.gitbook/assets/Proxy Architecture.png>)

When a user interacts with the contract, the Proxy contract doesn't "know" what functions and variables are there in the Impl. Calling functions in Impl is actually using **Delegatecall** from Proxy to Impl.

**Delegatecall's main feature**: code of callee is copied and run in the caller. I.e., the results of the computation, eg state variables, **are all stored in the Proxy**. And these variables are not indexed by name but by storage slot, so there is a problem of storage conflict.

|        | Proxy        | Impl            | Note       |
| ------ | ------------ | --------------- | ---------- |
| Slot 0 | address impl | <- bool var1    | Collision! |
| Slot 1 |              | <- bool var2    |            |
| Slot 2 |              | <- uint256 var3 |            |
| Slot 3 |              |                 |            |
|        |              |                 |            |
|        |              |                 |            |

In Proxy, a variable is used to record the address of the Impl contract, which has already conflicted in the above figure.&#x20;

To solve this, **EIP-1967** defined a storage slot location **IMPLEMENTATION\_SLOT** `= keccak256 ('eip1967.proxy.implementation' )) - 1`, and stores the impl address in this slot.

IMPLEMENTATION\_SLOT is a constant, and it won't occupy storage slot. Then, use solidity assembly to write the variable into the specified slot.

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

Then there's the collision is eliminated.

|             | Proxy        | Impl            | Note         |
| ----------- | ------------ | --------------- | ------------ |
| Slot 0      |              | <- bool var1    |              |
| Slot 1      |              | <- bool var2    |              |
| Slot 2      |              | <- uint256 var3 |              |
| Slot 3      |              |                 |              |
| ...         |              |                 |              |
| Slot Custom | address impl |                 | No collison! |
|             |              |                 |              |



### What happened in Audius

There were a customized variable `proxyAdmin` in its **multiple proxy contracts**, which caused the `Initalized` and `Initializing` variables in Impl that marked the initialization state to conflict with the `proxyAdmin` storage.

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

We ran a test on Rinkeby and found that both `initialized` and `initializing` was compromised(equals True) since deployment, as `address` type can cover multiple `bool` types(uint 8) with non-0 value.

With `initializing == true`, the modifier always allow initialisation.

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

In our test, if we remove the customized variable `proxyAdmin`, no re-initialisation was allowed. So the conclusion is clear.

## Summary

This attack happened due to re-initialisation was doable, which was caused by the improper understanding of proxy architecture.

Developers should read [OpenZeppelin Proxy Docs](https://docs.openzeppelin.com/contracts/4.x/api/proxy) before using its proxy module. Any customisation should be made with caution. `Ownable` instead of a customised variable should be used to manage access control. If you have to use a customised variable for special purposes, it should be implemented as EIP1967's storage guide.

## References

[https://docs.openzeppelin.com/contracts/4.x/api/proxy](https://docs.openzeppelin.com/contracts/4.x/api/proxy)

[https://eips.ethereum.org/EIPS/eip-1967](https://eips.ethereum.org/EIPS/eip-1967)
