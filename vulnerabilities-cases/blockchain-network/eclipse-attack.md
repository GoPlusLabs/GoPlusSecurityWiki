---
cover: >-
  https://images.unsplash.com/photo-1529788295308-1eace6f67388?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHwyfHxlY2xpcHNlfGVufDB8fHx8MTY1NjQwNDQ2Mw&ixlib=rb-1.2.1&q=80
coverY: 0
---

# Eclipse Attack

## Definition

Eclipse attacks are a special type of cyberattack for P2P networks, where an attacker creates an isolated malicious environment, with fake transactions/balances/etc., to deceive its visitors.

## Details

There are several ways to achieve an eclipse attack.

* **Client-side:** Isolate the target users by redirecting all relevant connections to the malicious node
* **Node-side:** Hack a well-known honest node and turn it into a malicious one, assuming most users won't check the data validity as it has a good reputation before
* **Hybrid:** Hack into both directions to make sure the maximum extent of the eclipse will be applied.

No matter which path you take, the **core idea** is to establish an isolated, manipulated and malicious network different from the main net, without being noticed in the first place by your target users.

The most common consequences of an eclipse attack in cryptocurrency projects include:

* **Double-spend attacks:** Once the victim is cut off from the network, the attacker may misdirect the victim into accepting a transaction that uses either an invalid input or the same input as another transaction that has already been validated on the legitimate network.
* **Miner power disruption:** Attackers can hide the fact that a block has been mined from an eclipsed miner, thereby misleading the victim into wasting time and computing power mining in a private blockchain. This way, the attacker is able to increase their relative hash rate within the network and bias the block-mining race in their favour. Furthermore, since an eclipsed miner is essentially blocked out from the legitimate network, attackers may launch eclipse attacks on multiple miners within a network in order to reduce the threshold required to launch a successful 51% attack on the entire network.



##
