---
cover: >-
  https://images.unsplash.com/photo-1529788295308-1eace6f67388?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHwyfHxlY2xpcHNlfGVufDB8fHx8MTY1NjQwNDQ2Mw&ixlib=rb-1.2.1&q=80
coverY: 0
---

# 日蚀攻击

## 定义

日蚀攻击是针对P2P网络的特殊攻击形式。攻击者创建与真实网络隔绝的恶意网络环境，并在其中发布虚假的交易、转账、余额等信息，来欺骗网络访问者。

## 细节

日蚀攻击可以通过下列途径完成：

* **客户端侧：**通过重定向所有相关通信至恶意节点，来隔绝目标用户
* **节点侧:** 入侵一个知名的公共节点并将其黑化，并假设基于之前该节点的名誉，大部分用户不会检查其数据真实性
* **混合**：在上述两个方向上均进行入侵活动，确保日蚀覆盖程度最大化

No matter which path you take, the **core idea** is to establish an isolated, manipulated and malicious network different from the main net, without being noticed in the first place by your target users.

The most common consequences of an eclipse attack in cryptocurrency projects include:

* **Double-spend attacks:** Once the victim is cut off from the network, the attacker may misdirect the victim into accepting a transaction that uses either an invalid input or the same input as another transaction that has already been validated on the legitimate network.
* **Miner power disruption:** Attackers can hide the fact that a block has been mined from an eclipsed miner, thereby misleading the victim into wasting time and computing power mining in a private blockchain. This way, the attacker is able to increase their relative hash rate within the network and bias the block-mining race in their favour. Furthermore, since an eclipsed miner is essentially blocked out from the legitimate network, attackers may launch eclipse attacks on multiple miners within a network in order to reduce the threshold required to launch a successful 51% attack on the entire network.

##
