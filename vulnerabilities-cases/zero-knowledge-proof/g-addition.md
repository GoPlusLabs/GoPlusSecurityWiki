---
cover: >-
  https://images.unsplash.com/photo-1460552181709-52ca8cb46336?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw0fHxpZGVudGljYWx8ZW58MHx8fHwxNjU3MDk4OTgy&ixlib=rb-1.2.1&q=80
coverY: -231.81454836131098
---

# Aliasing Attack

## Abstract

Some ZKP privacy applications need to prevent double spending by **nullifier**, which is bound within a valid proof in a ZK way so that it can be examined in the Verifier and marked as "used" after a corresponding action is completed to avoid double spending.

But this can be bypassed by the arithmetic features of cyclic groups if the implementation is wrong.



Here are some examples:

[https://github.com/semaphore-protocol/semaphore/issues/16](https://github.com/semaphore-protocol/semaphore/issues/16)

[https://github.com/eea-oasis/baseline/issues/34](https://github.com/eea-oasis/baseline/issues/34)

[https://github.com/semaphore-protocol/semaphore/pull/96/](https://github.com/semaphore-protocol/semaphore/pull/96/)

## Background

To reduce gas required by some ZKP schemes, Ethereum defined two precompile contracts for addition and multiplication operations of the elliptic curve **alt\_bn128** in [EIP-196](https://eips.ethereum.org/EIPS/eip-196).

Curve **alt\_bn128** is defined as:

$$
(x,y)\in (\mathbb F_p)^2\space\space |\space \space \lbrace y^2 ≡ x^3 + 3(mod \, p)\rbrace \cup \lbrace (0,0) \rbrace
$$

$$
\\p= 21888242871839275222246405745257275088696311157297823662689037894645226208583
$$

In the finite field `F_p`, there is a cyclic subgroup of order `q` based on generator `P1(1,2)`:

$$
q = 21888242871839275222246405745257275088548364400416034343698204186575808495617
$$

In that cyclic subgroup, there is

$$
x \in \mathbb F_q\, ,n \in \N \, | \, x+nq≡x(mod\,p)
$$

Namely, we can regard `q` as zero in the finite field operation, adding any multiples of `q` to the same point always return itself.

Since we usually use `uint256` in related calculations, which has a maximum `M`, there exist `N` aliases for a given input argument `A`:

$$
N =⌊(M-A)/q⌋
$$

That is to say, if `A` is the **nullifier**, you can construct at most `N` aliases, they are different natural numbers but will output identical results in the cyclic group. Hence, the double spending proof is invalid here, even can be abused to triple or quadruple spending.



This kind of attack was first revealed in the privacy layer project **Semaphore,** and its course can be traced back to 2017 when **Christian Reitwiessner** wrote such an unsafe example for implementation. Unfortunately many zkSNARKS libs like `snarkjs` and `ethsnarks` followed this example.

## Solution

The solution is simple. Just restrict the inputs should be less than `q` (`snark_scalar_field`).

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

## References

[https://paper.seebug.org/995/](https://paper.seebug.org/995/)

[https://ethereum.github.io/yellowpaper/paper.pdf](https://ethereum.github.io/yellowpaper/paper.pdf)

[https://ecips.ethereumclassic.org/ECIPs/ecip-1025](https://ecips.ethereumclassic.org/ECIPs/ecip-1025)



