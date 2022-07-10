---
cover: >-
  https://images.unsplash.com/photo-1547756536-cde3673fa2e5?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw4fHxicmlkZ2V8ZW58MHx8fHwxNjU2NDA1MDI2&ixlib=rb-1.2.1&q=80
coverY: 0
---

# Cross-chain Bridge

Cross-chain bridge is a system that passes information(asset cross-chain transfer, message call, etc.) between two or more chains.

From finality and security perspective, there are three kinds of bridges:

* Witness bridge, which adopts a group of witnesses as judges to determine whether a cross-chain transaction has happened and pass the info to other chains. Most bridges utilise this model by multi-sig vault management.
* Relay bridge, relayers relay validity proof(block header, etc.) from one chain to another. There are smart contracts that act like light clients on both chains to validate the proof. This kind could be attacked by Parallel Universe Attack if the connected chains are not covered by the same set of blockchain nodes/validators.
* Autonomy bridge, if one chain is subordinate to another(eg. rollup L2 and its L1), and its bridge(usually the official one) is a part of the control system, we name it an autonomy bridge since it has the power to determine the finality only by the cross-chain info within itself, without any consensus from external judges.
