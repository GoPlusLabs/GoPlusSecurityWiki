---
cover: >-
  https://images.unsplash.com/photo-1534224039826-c7a0eda0e6b3?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHwyfHxjb25uZWN0aW9ufGVufDB8fHx8MTY1ODg5NTAwNQ&ixlib=rb-1.2.1&q=80
coverY: 0
---

# Proxy Contract

A proxy contract is a contract which delegates calls to another contract. To interact with the actual contract you have to go through the proxy, and the proxy knows which contract to delegate the call to (the target).

A proxy pattern is used when you want upgradability for your contracts. This way the proxy contract stays immutable, but you can deploy a new contract behind the proxy contract - simply change the target address inside the proxy contract.
