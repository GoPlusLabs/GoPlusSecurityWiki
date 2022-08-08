# Nomad

## 摘要

Nomad是[Optics protocol](https://github.com/celo-org/celo-monorepo) (OPTimistic Interchain Communication)的实现与扩展，也即一个乐观式的跨链桥。

被攻击是由于错误的初始化参数。

| 状态       | 已修复                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------ |
| 类型         | 合约，跨链桥                                                               |
| 日期         | Aug 2, 2022                                                                                |
| 来源       | [@foobar](https://twitter.com/0xfoobar/status/1554269047176548353)                         |
| 直接损失  | \~$90M                                                                                     |
| 项目仓库 | [https://github.com/nomad-xyz/nomad-monorepo](https://github.com/nomad-xyz/nomad-monorepo) |



## 攻击路径和细节

1\)在[该tx](https://tools.blocksec.com/tx/eth/0xc4938e6f6368061194d076d44f73a8cae3a318b1ee7cf8b026abe10b7c206c2a)中, 黑客调用了Replica.sol中的process()。只要能通过这三个require，指定操作就会由`NomadBridge.handle()`执行。第一个和第三个显然，我们看下第二个require：`acceptableRoot(messages[_messageHash])`. 

```
function process(bytes memory _message) public returns (bool _success) {
    // ensure message was meant for this domain
    bytes29 _m = _message.ref(0);
    require(_m.destination() == localDomain, "!destination");
    // ensure message has been proven
    bytes32 _messageHash = _m.keccak();
    require(acceptableRoot(messages[_messageHash]), "!proven");
    // check re-entrancy guard
    require(entered == 1, "!reentrant");
    entered = 0;
    // update message status as processed
    messages[_messageHash] = LEGACY_STATUS_PROCESSED;
    // call handle function
    IMessageRecipient(_m.recipientAddress()).handle(
        _m.origin(),
        _m.nonce(),
        _m.sender(),
        _m.body().clone()
    );
    // emit process results
    emit Process(_messageHash, true, "");
    // reset re-entrancy guard
    entered = 1;
    // return true
    return true;
}
```



`messages[_messageHash] = 0x0`, 因为这个message是黑客捏造的在合约记录中并没有，所以在mapping中默认是0。而两个LEGACY值=1或2，与这里无关。下面是`confirmAt[_root]`，这个值只要非0且小于当前区块时间，就可以过了。那么`confirmAt[0x0]`的值是什么？
```
function acceptableRoot(bytes32 _root) public view returns (bool) {
    // this is backwards-compatibility for messages proven/processed
    // under previous versions
    if (_root == LEGACY_STATUS_PROVEN) return true;
    if (_root == LEGACY_STATUS_PROCESSED) return false;

    uint256 _time = confirmAt[_root];
    if (_time == 0) {
        return false;
    }
    return block.timestamp >= _time;
}
```

错误的初始化值：`confirmAt[_committedRoot] = 1`，而他们初始化时让`_committedRoot = 0x0`。那么所以`confirmAt[0x0] = 1`，上面的检查就能通过了
```
function initialize(
    uint32 _remoteDomain,
    address _updater,
    bytes32 _committedRoot,
    uint256 _optimisticSeconds
) public initializer {
    __NomadBase_initialize(_updater);
    // set storage variables
    entered = 1;
    remoteDomain = _remoteDomain;
    committedRoot = _committedRoot;
    // pre-approve the committed root.
    confirmAt[_committedRoot] = 1;
    _setOptimisticTimeout(_optimisticSeconds);
}
```

### 公开洗劫

这意味着所有人都可以随便捏造交易，也可以复制黑客的改一下接收地址。这也是为什么池子被掏空这么快。据报道有至少70个地址参与了这次公开洗劫。
