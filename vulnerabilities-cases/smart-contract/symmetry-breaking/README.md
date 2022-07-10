---
cover: >-
  https://images.unsplash.com/photo-1592813790189-2a42ebd9a863?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw1fHxzeW1tZXRyeXxlbnwwfHx8fDE2NTY0MDQwNTM&ixlib=rb-1.2.1&q=80
coverY: 0
---

# 对称性破缺

## 定义

对于一组有对称性的行为，某些限制条件在某一端遗失了，造成了安全对称性的破缺，可能会被攻击者利用。

## 对称性示例

* 充值（开始计息）- 提款（结束计息）
* 质押 - 解押
* 入住 - 退房&#x20;

### 对称性破缺

在这些成对的行为中，如质押-解押，如果在解押时忘记了一些检查（如，停止质押奖励），则系统会立马变得**不对称**，也就是说有大量的奖励会分配给那些已经退出质押的人，他们并没有资格获得奖励。
