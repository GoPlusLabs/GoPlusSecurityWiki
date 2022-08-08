# Nomad

## Abstract

Nomad is an implementation and extension of the [Optics protocol](https://github.com/celo-org/celo-monorepo) (OPTimistic Interchain Communication), i.e. an optimistic cross-chain bridge.

It was hacked due to wrong initialisation parameters.

| Stauts       | Fixed                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------ |
| Type         | Contract, Cross-chain Bridge                                                               |
| Date         | Aug 2, 2022                                                                                |
| Source       | [@foobar](https://twitter.com/0xfoobar/status/1554269047176548353)                         |
| Direct Loss  | \~$90M                                                                                     |
| Project Repo | [https://github.com/nomad-xyz/nomad-monorepo](https://github.com/nomad-xyz/nomad-monorepo) |



## Attack Vectors & Details

1\)In [this tx](https://tools.blocksec.com/tx/eth/0xc4938e6f6368061194d076d44f73a8cae3a318b1ee7cf8b026abe10b7c206c2a), the hacker just called process() in Replica.sol. Once you passed these three requires, the specified operations will be processed by NomadBridge.handle().&#x20;

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



All the requires passed. The first and third ones are obvious, so check the second one: `acceptableRoot(messages[_messageHash])`. `messages[_messageHash] = 0x0`, because the message was forged by the hacker(non-existent in this contract’s history).

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

In a mapping, it will be 0 by default. `LEGACY_XXXX = 1 or 2`, irrelevant here. Next is `confirmAt[_root]`, as long as it `!= 0` and `< current block time` then the check will pass. So what’s the value of `confirmAt[0x0]` ?

Here is the wrong initialisation param: confirmAt\[\_committedRoot] = 1. They passed \_committedRoot = 0x0 while initialising the contract. So confirmAt\[0x0] = 1. Check passed.

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

### Public Loot

Everyone can copy\&paste the hacker's tx data and modify the receiver's address to benefit their own. It was reported there were at least 70 addresses did this, the public loot.

