# Metamask Demonic漏洞

## 摘要

10.11.3版本之前的MetaMask存在Demonic漏洞，黑客有可能以此访问到用户的助记词。在之前版本的MetaMask中，BIP39助记词显示在输入框中，而Firefox和Chromium等浏览器会将该输入框中的明文直接保存在硬盘中以支持会话恢复功能。

有趣的是Demonic一词是双关的，既可以表示邪恶的，也可以理解为Mnemonic(助记词)的反语。

| 状态   | 已修复                                                                                              |
| ---- | ------------------------------------------------------------------------------------------------ |
| 类型   | 客户端                                                                                              |
| 日期   | Jun 10, 2022                                                                                     |
| 来源   | [Halborn](https://halborn.com/disclosures/demonic-vulnerability/)                                |
| 项目仓库 | [https://github.com/MetaMask/metamask-extension](https://github.com/MetaMask/metamask-extension) |

## 细节

### 会话恢复

Chrome（和某些浏览器）有恢复意外关闭的会话功能（如页面，窗口等）。

```
chrome.sessions.restore(
  sessionId?: string,
  callback?: function,
)
```

`sessions` 对象包含了会话的相关信息，包括输入框中的 `/Users/"$USER"/Library/Application Support/Google/Chrome/Default/Sessions`.F

## 解决方案

将助记词输入框变为一词一个. 每个框均设置为 `type="password"` , 该属性标记的输入框中的内容不会在恢复会话中使用。

![Remove the single text field and turned it into multiple ones with type=password](../../.gitbook/assets/before\_after.jpg)

## 参考

[https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32969](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32969)

[https://halborn.com/disclosures/demonic-vulnerability/](https://halborn.com/disclosures/demonic-vulnerability/)

[https://github.com/MetaMask/metamask-extension/pull/14016](https://github.com/MetaMask/metamask-extension/pull/14016)
