# Input Aliasing

## Abstract

Some ZKP privacy applications need to prevent double spending by **nullifier**, which is bound within a valid proof in a ZK way so that it can be examined in the Verifier and marked as "used" after a corresponding action is completed to avoid double spending.

But this can be bypassed by the arithmetic features of cyclic groups if the implementation is wrong.



Here are some examples:

[https://github.com/semaphore-protocol/semaphore/issues/16](https://github.com/semaphore-protocol/semaphore/issues/16)

[https://github.com/eea-oasis/baseline/issues/34](https://github.com/eea-oasis/baseline/issues/34)

[https://github.com/semaphore-protocol/semaphore/pull/96/](https://github.com/semaphore-protocol/semaphore/pull/96/)

## Cryptography Background

To reduce gas required by some ZKP schemes, Ethereum defined two precompile contracts for addition and multiplication operations of the elliptic curve **alt\_bn128** in [EIP-196](https://eips.ethereum.org/EIPS/eip-196).

Curve **alt\_bn128** is defined as:

$$
(x,y)\in (\mathbb F_p)^2\space\space |\space \space \lbrace y^2 ≡ x^3 + 3(mod \, P)\rbrace \cup \lbrace 0 \rbrace
$$

$$
\\p= 21888242871839275222246405745257275088696311157297823662689037894645226208583
$$







```
 uint256 snark_scalar_field = 21888242871839275222246405745257275088548364400416034343698204186575808495617;
    VerifyingKey memory vk = verifyingKey();
    require(input.length + 1 == vk.IC.length, "verifier-bad-input");
    Pairing.G1Point memory vk_x = Pairing.G1Point(0, 0);
    for (uint256 i = 0; i < input.length; i++) {
       // Check for field arithmetic
      require(input[i] < snark_scalar_field, "verifier-gte-snark-scalar-field");
      vk_x = Pairing.addition(vk_x, Pairing.scalar_mul(vk.IC[i + 1], input[i]));
    }
```



Point coordinates `input[i]` should < G, or cyclic operation will be exploited.
