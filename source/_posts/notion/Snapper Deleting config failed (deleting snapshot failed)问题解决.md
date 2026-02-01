---
created: 2026-01-31 07:21:00
categories:
  - linux
tags:
  - linux
updated: 2026-02-01 10:45:00
date: 2026-02-01 00:00:00
title: Snapper Deleting config failed (deleting snapshot failed)问题解决
id: 2f94ee3d-e06e-80c1-9a64-c6fdced40d57
---

### **问题描述**

当尝试删除 Snapper 配置文件时，系统提示以下错误信息：

`Deleting config failed (deleting snapshot failed)`

**错误日志分析**

检查  `/var/log/snapper.log`  日志文件，发现以下错误信息：

```javascript
Sat Jan 31 15:19:15 2026 <2> FileUtils.cc(SDir):88 THROW: open failed path:/home/.snapshots errno:2 (No such file or directory)
Sat Jan 31 15:19:15 2026 <2> Snapshot.cc(initialize):427 CAUGHT: open failed path:/home/.snapshots errno:2 (No such file or directory)
```

日志清晰地显示了  `open failed path:/home/.snapshots errno:2 (No such file or directory)`。这表明  `/home`  目录下缺少  `.snapshots`  子卷，导致 Snapper 在尝试删除相关快照时找不到其存储位置，从而无法完成配置删除操作。

解决办法，在 `home` 目录下面创建 `.snapshots` subvolume

### **解决方案**

要解决此问题，需要在  `/home`  目录下创建  `.snapshots` Btrfs 子卷。创建完成后，即可成功删除 Snapper 配置。

1. **创建** **`.snapshots`** **子卷：**

   ```bash
   sudo btrfs subvolume create /home/.snapshots
   ```

2. **重新尝试删除 Snapper 配置：**

   ```bash
   sudo snapper -c home_cfg delete-config
   ```
