# Cream Finance

## Abstract

Cream Finance is a decentralised lending protocol. The most notorious thing about it is it had been attacked by Flash Loan attack three times. We pick one of the attacks to analyse.

The main vulnerability is it used price feed from oracle directly, which could be manipulated by large capital borrowed from flash loan.

| Status       | Fixed                                                      |
| ------------ | ---------------------------------------------------------- |
| Type         | Flashloan                                                  |
| Date         | Oct 27, 2021                                               |
| Source       | -                                                          |
| Direct Loss  | $115 million                                               |
| Project Repo | [https://github.com/CreamFi/](https://github.com/CreamFi/) |

## Flash Loan Steps

Exploiter's address

[https://etherscan.io/address/0x24354d31bc9d90f62fe5f2454709c32049cf866b\
](https://etherscan.io/address/0x24354d31bc9d90f62fe5f2454709c32049cf866b)

Attack Tx

[https://etherscan.io/tx/0x0fe2542079644e107cbf13690eb9c2c65963ccb79089ff96bfaf8dced2331c92](https://etherscan.io/tx/0x0fe2542079644e107cbf13690eb9c2c65963ccb79089ff96bfaf8dced2331c92)

1. Borrow 500M DAI from DssFlash and covert to 450M yDAI
2. Transfer yDAI to Curve.fi pool and convert it to 447M Curve.fi yDAI/yUSDC/yUSDT/yTUSD token and 446M yUSD.
3. Transfer 446M yUSD to Cream and get 22.3B crYUSD; Transfer 447M Curve.fi yDAI/yUSDC/yUSDT/yTUSD token into strategy pool.
4. Exploiter Contract 2 borrowed 524K WETH from Aave flash loan then transferred 6K WETH to Exploiter Contract 1.
5. Borrow 24.95M crETH with the remaining 518K WETH.
6. Exploiter 2 borrowed 446M yUSD and swapped them for 22.3B crYUSD then transferred Exploiter 1.

... The whole process is too long to elaborate on, if you're interested you can find the details in the transaction record. These steps were aimed to tilt the exchange rate significantly.

Finally, by manipulating price differences, the attack borrowed a large amount of capital and then repaid flash loan, the remaining was his profit.

## Summary

* Price feed from oracle
* Oracle calculated share by the asset pool usage
* Manipulate share to manipulate price

