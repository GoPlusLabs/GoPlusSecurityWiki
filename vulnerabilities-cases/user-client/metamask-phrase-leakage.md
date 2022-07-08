# Metamask Demonic Vulnerability

## Abstract

MetaMask before 10.11.3 might allow an attacker to access a user's secret recovery phrase because an `input field` is used for a BIP39 mnemonic, and Firefox and Chromium **save such fields to disk without encryption** in order to support the Restore Session feature, aka the Demonic issue.

| Status       | Fixed                                                                                            |
| ------------ | ------------------------------------------------------------------------------------------------ |
| Type         | User Client                                                                                      |
| Date         | Jun 10, 2022                                                                                     |
| Source       | [Halborn](https://halborn.com/disclosures/demonic-vulnerability/)                                |
| Project Repo | [https://github.com/MetaMask/metamask-extension](https://github.com/MetaMask/metamask-extension) |

## Details

### Restore Session

Chrome(and some other browsers) can restore sessions(pages, windows, etc.) closed unintentionally by calling

```
chrome.sessions.restore(
  sessionId?: string,
  callback?: function,
)
```

The `sessions` object contains some information, including unencrypted plain text from text input field, which is stored at `/Users/"$USER"/Library/Application Support/Google/Chrome/Default/Sessions`.

### Fix

Split the secret recovery phrase input field into one field per word. Each word uses a `type="password"` field, which cannot be stored by the restore sessions feature.

![Remove the single text field and turned it into multiple ones with type=password](../../.gitbook/assets/before\_after.jpg)

## References

[https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32969](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32969)

[https://halborn.com/disclosures/demonic-vulnerability/](https://halborn.com/disclosures/demonic-vulnerability/)

[https://github.com/MetaMask/metamask-extension/pull/14016](https://github.com/MetaMask/metamask-extension/pull/14016)
