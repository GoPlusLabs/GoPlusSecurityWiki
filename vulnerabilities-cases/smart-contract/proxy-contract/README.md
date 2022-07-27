---
cover: >-
  https://images.unsplash.com/photo-1534224039826-c7a0eda0e6b3?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHwyfHxjb25uZWN0aW9ufGVufDB8fHx8MTY1ODg5NTAwNQ&ixlib=rb-1.2.1&q=80
coverY: 0
---

# 代理合约

代理合约使用delegatecall调用其实现合约。为了与实现合约进行沟通，需要经过代理，代理需要了解其实现合约的地址。

代理模式用于合约的可升级性。使用代理合约架构，代理合约是不可变的，但新部署的实现合约是可变的，只需要在代理中修改地址。