# Metamask Denomic漏洞

## 摘要

在10.11.3之前，MetaMask对BIP39助记词使用的输入框有漏洞。Firefox、Chromium等浏览器为了实现**会话恢复**功能会直接将内容以明文形式存储在磁盘，可以让攻击者能够访问到助记词。

| 状态       | 已修复                                                                                            |
| ------------ | ------------------------------------------------------------------------------------------------ |
| 类型         | 客户端                                                                                      |
| 日期         | Jun 10, 2022                                                                                     |
| 来源       | [Halborn](https://halborn.com/disclosures/demonic-vulnerability/)                                |
| 项目仓库 | [https://github.com/MetaMask/metamask-extension](https://github.com/MetaMask/metamask-extension) |

## 细节

### 恢复会话

Chrome和某些浏览器能够恢复意外关闭的会话（页面，窗口等）

```
chrome.sessions.restore(
  sessionId?: string,
  callback?: function,
)
```

`sessions`包含了一些必要信息，包括会被邪路的助记词，均以不加密的形式存储在 `/Users/"$USER"/Library/Application Support/Google/Chrome/Default/Sessions`.

### 修复方法

之前使用了一整个框来显示助记词。修复后，一个助记词一个输入框，每个框都有`type="password"`属性，该属性阻止了浏览器存放其中的内容用于会话恢复。

![Remove the single text field and turned it into multiple ones with type=password](../../.gitbook/assets/before\_after.jpg)

## 参考

[https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32969](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32969)

[https://halborn.com/disclosures/demonic-vulnerability/](https://halborn.com/disclosures/demonic-vulnerability/)

[https://github.com/MetaMask/metamask-extension/pull/14016](https://github.com/MetaMask/metamask-extension/pull/14016)
