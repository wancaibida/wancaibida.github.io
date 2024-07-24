---
created: 2024-05-17 03:10:00
categories:
  - linux
tags:
  - linux
  - macos
updated: 2024-05-17 09:21:00
date: 2024-05-17 00:00:00
title: Dropbox Operation not permitted 问题修复
id: 295c8d78-eb06-48e3-ba84-1559a6f45d31
---

Dropbox 在 macOS Ventura 中将同步目录从 `~/Dropbox` 移至 `/Users/xxx/Library/CloudStorage/Dropbox` 后，使用 iTerm `ls` 命令访问该目录会提示 `Operation not permitted` 错误。这是因为 iTerm 对新目录没有访问权限。

解决方法：打开 `system settings` → `Privacy&Security` → `Full Disk Access` → `iTerm`，为 iTerm 启用访问权限即可。

这里还有一个问题就是如果你将 `~/.ssh` 目录符号链接到同步文件夹会有权限问题，同步文件夹的上级目录的权限是不一样的。
