---
created: 2025-12-25 01:42:00
categories:
  - linux
tags:
  - linux
updated: 2025-12-31 11:59:00
date: 2025-12-30 00:00:00
title: 解决CopyQ在KDE环境下全局快捷键失效的问题
id: 2d44ee3d-e06e-801e-9209-f276493bf791
---

Copyq 在 kde 环境下全局快捷键不起作用，可能的原因是 copy 自启动之后与全局快捷键有冲突，具体现象是在 copyq 开启 auto start 之后，在 shortcut 系统设置里会出现两个 copyq 的快捷键设置，一个在 Applications 下面，一个在 System Services 。

这两个程序都会去注册全局快捷键，然后就会提示下面的错误

```javascript
Dec 25 09:16:16 ThinkPad copyq[2156]: Warning: [qt.qpa.services] Failed to register with host portal QDBusError("org.freedesktop.portal.Error.Failed", "Could not register app ID: Connection already associated with an application ID")
```

解决办法：

1. 在 copyq 设置了把全局快捷键删除，但是保留开启 auto start
2. 在 kde 设置里面把 Applications 下面的 copyq 删除
3. 然后添加一个 command 绑定到全局快捷键，command 内容如下：

   ```javascript
   /usr/bin/copyq -e "toggle()"
   ```

参考：

[https://copyq.readthedocs.io/en/latest/faq.html#why-do-global-shortcuts-not-work](https://copyq.readthedocs.io/en/latest/faq.html#why-do-global-shortcuts-not-work)
