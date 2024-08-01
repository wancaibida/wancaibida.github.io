---
created: 2024-08-01 02:10:00
categories:
  - life
updated: 2024-08-01 02:54:00
date: 2024-08-01 00:00:00
title: infuse刮削逻辑研究
id: 058d4de3-c2f7-4dde-aa51-6e9a5beddddb
---

我最近研究了 Infuse 的刮削逻辑，发现它的[官方文档](https://www.notion.so/21e3e12d36c54f448ef05cf2f34ec424)写得不太清楚。我用 Infuse + 阿里云盘的方式测试了一下，总结如下：

# 剧集

```shell
folder1/foder2/file1.mkv
folder1/foder2/file2.mkv

```

- 对于剧集来说，`folder1` 和 `folder2` 的命名并不重要，Infuse 并不关心它们的名称。关键在于 `file1` 的文件名。如果你希望 Infuse 将 `folder2` 识别为一个剧集，那么 `file1` 和 `file2` 的文件名必须遵循 `剧集名称.第几季.第几集.后缀名` 的格式，例如 `Peppa.Pig.S01.E01.mkv` 表示《小猪佩奇》第一季第一集。
- 官方文档显示剧集信息源自 [TMDB](https://www.themoviedb.org/)，但实际测试下来似乎并非如此。如有剧集名称疑问，可手动修改 file1 元数据，并参考搜索结果中的剧集名称进行命名。
- 如需批量重命名文件，可以使用 Mountain Duck + Alist 挂载阿里云盘，然后利用 macOS 的批量重命名功能。Mountain Duck + 阿里云盘官方 WebDAV 无法重命名文件，会返回 403 错误。

# 排除文件夹

[文档](https://support.firecore.com/hc/en-us/articles/4405044108183-Excluding-Files-and-Folders)中的方法测试了下没有效果，不知到是啥原因，只能把文件夹移动到别的地方。
