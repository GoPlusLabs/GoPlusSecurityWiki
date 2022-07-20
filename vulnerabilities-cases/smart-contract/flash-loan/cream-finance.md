# Cream Finance

## 摘要

Cream Finance是一个去中心化借贷协议。该协议最著名的是，历史上遭受过三次闪电贷攻击。我们选取其中一次进行分析。

其主要的漏洞为直接使用了预言机提供的报价，而该报价可以被闪电贷借贷来的大资本进行操纵。

| 状态       | 已修复                                                      |
| ------------ | ---------------------------------------------------------- |
| 类型         | 闪电贷                                                  |
| 时间         | Oct 27, 2021                                               |
| 来源       | -                                                          |
| 直接损失  | $115 million                                               |
| 项目仓库 | [https://github.com/CreamFi/](https://github.com/CreamFi/) |

## 闪电贷步骤

攻击者地址

[https://etherscan.io/address/0x24354d31bc9d90f62fe5f2454709c32049cf866b\
](https://etherscan.io/address/0x24354d31bc9d90f62fe5f2454709c32049cf866b)

攻击Tx

[https://etherscan.io/tx/0x0fe2542079644e107cbf13690eb9c2c65963ccb79089ff96bfaf8dced2331c92](https://etherscan.io/tx/0x0fe2542079644e107cbf13690eb9c2c65963ccb79089ff96bfaf8dced2331c92)

1. 从DssFlash借500M DAI，转化为450M yDAI
2. 将yDAI转至Curve.fi池子中，得到447M Curve.fi yDAI/yUSDC/yUSDT/yTUSD token和446M yUSD
3. 将446M yUSD转移至Cream，获得22.3B crYUSD；将447MCurve.fi yDAI/yUSDC/yUSDT/yTUSD token转移至策略池
4. 攻击者合约2从Aave闪电贷借款524K WETH，然后将6K WETH转移至攻击者合约1
5. 用剩余的518K WETH借24.95M crETH
6. 攻击者2借了446M yUSD并转换成22.3B crYUSD然后转移至攻击者1.

... 整个过程过于冗长，不再赘述，可以在上述攻击Tx中查看详情。这些步骤的目的都是操纵汇率。

最终，通过操纵价格，攻击者借出了大量资金并归还了闪电贷，剩余部分为其利润。

## 总结

* 通过预言机获取价格
* 预言机通过资金池使用情况计算资金占比
* 操纵资金占比来操纵价格

