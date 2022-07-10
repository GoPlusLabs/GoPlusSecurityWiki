---
cover: >-
  https://images.unsplash.com/photo-1507457379470-08b800bebc67?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw1fHxwbGF5c3RhdGlvbnxlbnwwfHx8fDE2NTcwOTgzMjM&ixlib=rb-1.2.1&q=80
coverY: 556.3549160671463
---

# ECDSA随机数

## 摘要

在ECDSA过程中，需要一个随机数（也可能是确定性的不重复的哈希）来对消息签名。

如果你对不同的消息使用了同一个随机数，则私钥会暴露。

最出名的事件莫过于[Sony PlayStation 3 Hack](https://www.theguardian.com/technology/gamesblog/2011/jan/07/playstation-3-hack-ps3).

## 密码学背景

以下为几步**简化的**ECDSA过程:

### 密钥生成

* **私钥 d\_A**，由RNG生成的随机数
* **公钥 Q\_A:**

$$
Q_A=d_A*G \\ \text{G为曲线的生成元}
$$

### 签名

* 计算消息`M`的哈希值`h = hash(M)`
* 生成随机数`k`
* 计算随机点`R = kG`&#x20;
* 将点R的横坐标`R.x`记为`r`, 然后计算s

$$
s = k^{-1}(h+rd_A)(\bmod \,n)
$$

这样就得到了签名`(r,s)`.

### 验证签名

* 计算消息`M`的哈希值`h = hash(M)`
* 计算其逆

$$
s_1=s^{-1}(\bmod \,n)
$$

* 恢复在签名时使用的随机点

$$
R' = (hs_1) \times G + (r  s_1) \times Q_A
$$

* 检查是否有`R'.x == r`

## 攻击细节

显然，如果我们对不同的消息`M`使用了相同的`k`（也意味着`r`相同），可以通过下列几步解出私钥：

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

## 总结

* 使用ECDSA签名时绝不使用相同的随机数
* 或者，使用确定性ECDSA
* 开发者应该尽可能熟悉底层的密码学知识以避免类似攻击
