---
cover: >-
  https://images.unsplash.com/photo-1562065540-efa93744ed71?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw0fHxjdXJ2ZXxlbnwwfHx8fDE2NTcwOTgzNjA&ixlib=rb-1.2.1&q=80
coverY: 0
---

# Ed25519

## 摘要

EdDSA算法本身是安全的，但有些EdDSA库的实现是不安全的，有可能会导致私钥泄露。

## 密码学背景

**Edwards曲线数字签名算法** (**EdDSA**)是一种[数字签名](https://en.wikipedia.org/wiki/Digital\_signature)方案，使用了[Schnorr签名](https://en.wikipedia.org/wiki/Schnorr\_signature)的变种，基于[扭曲Edwards曲线](https://en.wikipedia.org/wiki/Twisted\_Edwards\_curve)。

**EdDSA**签名算法及其变种**Ed25519**和**Ed448**在[RFC 8032](https://tools.ietf.org/html/rfc8032)有完整的技术性描述。

### 私钥: k

k = RNG生成的随机数

### 衍生的一个整数: **a**

计算私钥的摘要：

$$
H(k)=(h_0,h_1,...,h_{2b-1})
$$

然后计算整数`a`:

$$
a = 2^{b-2} + \sum_{\substack 3⩽i⩽b-3} 2^ih_i \in \lbrace {2^{b-2},2^{b-2}+8,...,2^{b-1}-8} \rbrace
$$

后面你会发现，这个`a`也是一个不能泄露的数，它基本等同于私钥。

### 公钥: A

$$
A = aB
$$

`B`为曲线的基点

### 签名生成: (R,s)

对于消息`M`:

$$
r=H(h_b,...,h_{2b-1},M) \in 0,1,...2^{2b}-1
$$

$$
R=rB
$$

$$
s=(r+H(R,A,M)a)\bmod l
$$

`l`是点`B`生成的子群的阶。

现在我们得到了签名`(R,s)`. 显然，如果你知道了`a`则可以计算任意签名, 等同于知道了私钥`k`。

### 验证签名

$$
if\space 8sB == 8R+8H(R,A,M)A
$$

​
## 不安全的实现

想象一下这种函数实现:&#x20;

```
//PSEUDO CODE
func sign(message M, publicKey A){    
    R = rB
    S=(r+H(R,A,M)a) mod l
    return (R,S)    
}
```

该sign()函数允许调用者输入任意消息和公钥。如果有人使用了相同的`M`但不同的`A`，则他可以计算出整数`a`，私钥`k`的等价物。
 
$$
a=(S-S')[H(R,A,M)-H(R,A',M)]^{-1} \bmod l
$$

有很多库有类似的不安全实现，[查看列表](https://github.com/MystenLabs/ed25519-unsafe-libs).

## 总结

这是对Ed25519对不安全实现，虽然并不意味着一定会出问题，但还是要尽可能规范。.&#x20;

开发者应该尽可能熟悉底层的密码学知识以避免类似攻击。

## 参考

[https://twitter.com/kostascrypto/status/1535579208960790528](https://twitter.com/kostascrypto/status/1535579208960790528)

[https://datatracker.ietf.org/doc/html/rfc8032](https://datatracker.ietf.org/doc/html/rfc8032)



