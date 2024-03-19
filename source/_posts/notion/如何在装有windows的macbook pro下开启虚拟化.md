---
created: 2024-03-08 08:20:00
categories:
  - linux
tags:
  - windows
updated: 2024-03-19 07:22:00
date: 2024-03-16 00:00:00
title: 如何在装有windows的macbook pro下开启虚拟化
id: 5f2bdb92-c2c1-498c-b5a3-0d6a2adb2b42
---

家里有一台常年吃灰的 2015 款 MacBook Pro，最近想体验下 WSL2，就打算废物利用，将这台 Mac 格式化安装了 Windows，安装完成后发现未开启虚拟化，由于我只安装了 windows 系统，没法通过 boot camp 的方式来开启虚拟化，研究了下可以通过安装 rEFInd 来开启虚拟化：

<div style="width: 100%; margin-top: 4px; margin-bottom: 4px;"><div style="display: flex; background:white;border-radius:5px"><a href="https://www.rodsbooks.com/refind/installing.html#windows"target="_blank"rel="noopener noreferrer"style="display: flex; color: inherit; text-decoration: none; user-select: none; transition: background 20ms ease-in 0s; cursor: pointer; flex-grow: 1; min-width: 0px; flex-wrap: wrap-reverse; align-items: stretch; text-align: left; overflow: hidden; border: 1px solid rgba(55, 53, 47, 0.16); border-radius: 5px; position: relative; fill: inherit;"><div style="flex: 4 1 180px; padding: 12px 14px 14px; overflow: hidden; text-align: left;"><div style="font-size: 14px; line-height: 20px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; min-height: 24px; margin-bottom: 2px;">The rEFInd Boot Manager: Installing and Uninstalling rEFInd</div><div style="font-size: 12px; line-height: 16px; color: rgba(55, 53, 47, 0.65); height: 32px; overflow: hidden;"></div><div style="display: flex; margin-top: 6px; height: 16px;"><img src=""style="width: 16px; height: 16px; min-width: 16px; margin-right: 6px;"><div style="font-size: 12px; line-height: 16px; color: rgb(55, 53, 47); white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">https://www.rodsbooks.com/refind/installing.html#windows</div></div></div></a></div></div>

修改配置：

1. 打开 `refind`文件夹，找到 `refind.conf-sample`文件，重命名为 `refind.conf`
2. 用文本编辑器打开它，搜索 `enable_and_lock_vmx`，把它的值改为 `true`，并且注意把前面的 `#` 号去掉
