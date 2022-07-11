---
cover: >-
  https://images.unsplash.com/photo-1460552181709-52ca8cb46336?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw0fHxpZGVudGljYWx8ZW58MHx8fHwxNjU3MDk4OTgy&ixlib=rb-1.2.1&q=80
coverY: -231.81454836131098
---

# 别名攻击

## 摘要

有些ZKP隐私应用需要通过**nullifier**阻止双花，该nullifier以ZK的形式绑定在proof中，可以被验证者证明，并在相关动作结束后标记为“已使用”，来避免双花。

不过，如果应用实现有问题，这一点可以通过循环群的特性来绕开。

以下是几个例子：

[https://github.com/semaphore-protocol/semaphore/issues/16](https://github.com/semaphore-protocol/semaphore/issues/16)

[https://github.com/eea-oasis/baseline/issues/34](https://github.com/eea-oasis/baseline/issues/34)

[https://github.com/semaphore-protocol/semaphore/pull/96/](https://github.com/semaphore-protocol/semaphore/pull/96/)

## 背景

为了减少某些ZKP方案的gas消耗，以太坊为**alt\_bn128** in [EIP-196](https://eips.ethereum.org/EIPS/eip-196)曲线定义了加法和乘法的两个预编译合约。

曲线**alt\_bn128**的定义为：

$$
(x,y)\in (\mathbb F_p)^2\space\space |\space \space \lbrace y^2 ≡ x^3 + 3(mod \, p)\rbrace \cup \lbrace (0,0) \rbrace
$$

$$
\\p= 21888242871839275222246405745257275088696311157297823662689037894645226208583
$$

在有限域`F_p`中，基于生成元`P1(1,2)`的循环子群的阶为`q`。

$$
q = 21888242871839275222246405745257275088548364400416034343698204186575808495617
$$

在循环子群中，有

$$
x \in \mathbb F_q\, ,n \in \N \, | \, x+nq≡x(mod\,p)
$$

也即，`q`在有限域计算中可以被视为0，对任意相同点加上任意数量的`q`都会得到它自己。

由于我们在相关计算中一般使用`uint256`，其最大值为`M`，那么对于给定的参数`A`，有`N`个相同结果的别名：

$$
N =⌊(M-A)/q⌋
$$

也就是说，如果`A`是**nullifier**，你可以构建至多`N`个别名，它们是不同的自然数，但在循环群的运算中会得到相同的结果。因此，防止双花的证明就无效了，甚至可以三花四花五花。

此类型的攻击最初发现于隐私层应用**Semaphore**中，而其源头可以追溯至2017年**Christian Reitwiessner**写的一段不安全的样例实现。不幸的是，很多zkSNARKS的库如`snarkjs`和`ethsnarks`也都遵照了该样例。

## 解决方案

解决方案很简单，只需要限制输入值需要小于`q` (`snark_scalar_field`)。

```
    VerifyingKey memory vk = verifyingKey();
    require(input.length + 1 == vk.IC.length, "verifier-bad-input");
    Pairing.G1Point memory vk_x = Pairing.G1Point(0, 0);
    for (uint256 i = 0; i < input.length; i++) {
       // Check for field arithmetic less than q
      require(input[i] < snark_scalar_field, "verifier-gte-snark-scalar-field");
      vk_x = Pairing.addition(vk_x, Pairing.scalar_mul(vk.IC[i + 1], input[i]));
    }
```

## 参考

[https://paper.seebug.org/995/](https://paper.seebug.org/995/)

[https://ethereum.github.io/yellowpaper/paper.pdf](https://ethereum.github.io/yellowpaper/paper.pdf)

[https://ecips.ethereumclassic.org/ECIPs/ecip-1025](https://ecips.ethereumclassic.org/ECIPs/ecip-1025)



