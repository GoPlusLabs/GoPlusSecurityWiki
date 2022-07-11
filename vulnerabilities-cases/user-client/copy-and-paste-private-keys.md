---
cover: >-
  https://images.unsplash.com/photo-1527176930608-09cb256ab504?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw2fHxib29rfGVufDB8fHx8MTY1NzQxMzAzNw&ixlib=rb-1.2.1&q=80
coverY: 0
---

# 剪切板安全

## 摘要

一般用户使用多个钱包的常见操作为，复制粘贴私钥或助记词。对于跨设备的复制，中间还可能使用其他软件，如Telegram，微信等。

而该过程是非常危险的。所有的主流操作系统并不能保证剪切板安全。iOS稍微好些，但也还远远不够。

## Windows

Windows软件几乎有无限的权力来访问和操控剪切板。读写也都不会对用户有任何警示。

```
//.NET
// Namespace: System.Windows.Forms
Clipboard.GetText()
Clipboard.SetText(string)

// UWP
// Namespace: Windows.ApplicationModel.DataTransfer
Clipboard.SetContent(dataPackage)
Clipboard.GetContent()
```

## macOS

与Windows一样，任何桌面平台都能非常轻松地、无提示地读写剪切板。

```
// Cocoa
// Class NSPasteboard
func string(forType dataType: NSPasteboard.PasteboardType) -> String?
func setString(_ string: String, forType dataType: NSPasteboard.PasteboardType) -> Bool
```

## iOS

iOS在这个问题上表现地更安全一些。不过对普通用户而言还是无法甄别和阻止剪切板监控。

在前台或启用了后台模式的App（音乐，地图等）可以读取剪切板&#x20;

处于suspended状态的app（已经入后台且没有声明需要后台工作）无法实现这点。

自iOS 14后，当有应用获取了**通用剪切板**中来自其他app的内容后，系统会给用户**通知**。&#x20;

![Pasteboard Notification](../../.gitbook/assets/apollo-clipboard-alerts-9to5mac.jpg.webp)

```
// Cocoa Touch
// Class UIPasteboard
func data(forPasteboardType pasteboardType: String, inItemSet itemSet: IndexSet?) -> [Data]?
func setData(_ data: Data, forPasteboardType pasteboardType: String)
```

有些应用可能会使用`UIPasteboard.DetectionPattern`作为初始过滤器来减少该通知的频率。通过访问该结构，应用只会知道剪切板中的内容是否符合一定条件，但并不知道具体内容是什么，因此也不会有通知弹出。该过滤器比较简单，并没有正则表达式等高级功能能来匹配助记词或私钥。

### 接力

通过接力，当用户使用iOS, iPadOS和macOS设备并距离较近时，这些设备的剪切板是共享的。&#x20;

根据Apple的技术文档，我们可以认为接力在传输阶段是安全的。不过这并不能阻止设备端上的剪切板监控应用或恶意软件。

> When a user signs in to iCloud on a second Handoff-capable device, the two devices establish a Bluetooth Low Energy (BLE) 4.2 pairing out-of-band using APNs. The individual messages are encrypted much like messages in iMessage are. After the devices are paired, each device generates a symmetric 256-bit AES key that gets stored in the device’s [keychain](https://support.apple.com/zh-cn/guide/security/aside/sec1e14cf8d3/1/web/1). This key can encrypt and authenticate the BLE advertisements that communicate the device’s current activity to other iCloud paired devices using AES256 in GCM mode, with replay protection measures.
>
> The first time a device receives an advertisement from a new key, it establishes a BLE connection to the originating device and performs an advertisement encryption key exchange. This connection is secured using standard BLE 4.2 encryption as well as encryption of the individual messages, which is similar to how iMessage is encrypted. In some situations, these messages are sent using APNs instead of BLE. The activity payload is protected and transferred in the same way as an iMessage.

## Android

虽然在安卓中也有iOS一样清晰的应用生命周期，但至少有10种保活办法来绕过生命周期的限制，让应用一直处于能够监控其他事件的状态。而且安卓上也没有剪切板通知。

可以说，安卓上的情况**与桌面端一样糟糕**。

```
// Android, Java
pasteData = clipboard.getPrimaryClip().getItemAt(0).item.getText();
clipboard.setPrimaryClip(ClipData.newPlainText("simple text", "Hello, World!"););
```

## 建议

* 对应用**开发者**而言，如果你的应用允许用户复制粘贴私钥或助记词，推荐在他们粘贴完后清空剪切板。
* 对**用户**而言，如果你复制了你的敏感信息并在某处进行粘贴了，推荐你打开其他系统应用（当然，假设其是安全的，如记事本），并随便复制一些内容来清除剪切板。



