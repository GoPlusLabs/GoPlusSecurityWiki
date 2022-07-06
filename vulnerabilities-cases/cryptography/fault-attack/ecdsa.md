---
cover: >-
  https://images.unsplash.com/photo-1507457379470-08b800bebc67?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw1fHxwbGF5c3RhdGlvbnxlbnwwfHx8fDE2NTcwOTgzMjM&ixlib=rb-1.2.1&q=80
coverY: 556.3549160671463
---

# ECDSA random numbers

## Abstract

In the ECDSA signing process, a random number(or determined non-repeat hash number) is required to sign the message.

If you use the same random number for different messages, your private key will be exposed.

The most famous incident is [Sony PlayStation 3 Hack](https://www.theguardian.com/technology/gamesblog/2011/jan/07/playstation-3-hack-ps3).

## Cryptography Background

Several **simplified** steps in ECDSA:

### Key Generation

* **private key** **d\_A**, an integer generated from RNG
* **public key Q\_A:**

$$
Q_A=d_A*G \\ \text{G is the generator of the curve}
$$

### Signature

* Calculate hash `h = hash(M)` of the message `M`
* Generate a random number `k`
* Calculate the random point `R = kG`&#x20;
* Denote R's x coordinate `R.x` as `r`, then calculate s

$$
s = k^{-1}(h+rd_A)(\bmod \,n)
$$

Now we have the signature `(r,s)`.

### Verify Signature

* Calculate hash `h = hash(M)` of the message `M`
* Calculate the modular inverse

$$
s_1=s^{-1}(\bmod \,n)
$$

* Recover the random point used during the signing

$$
R' = (hs_1) \times G + (r  s_1) \times Q_A
$$

* Check if `R'.x == r`

## Attack Details

Apparently, if we use the same `k` (also means the same `r`) for different message `M`, the private key can be solved by the following steps:

$$
\begin{cases}
s = k^{-1}(h+rd_A)(\bmod \,n) \\
s' = k^{-1}(h'+rd_A)(\bmod \,n)
\end{cases}

\\
\implies
\\
\begin{cases}
k = (h-h')(S-S')^{-1} \\
d_A = (sk-h)r^{-1}
\end{cases}
$$

## Summary

* Never use the same random numbers when signing by ECDSA
* Or, use a deterministic ECDSA
* Developers should be familiar enough with the underlying cryptography to avoid similar attacks.
