---
title: npm 全局包备份与恢复
description: 升级 Node 版本前备份全局 npm 包列表，升级后一键恢复
type: how-to
tags:
  - npm
  - node
  - windows
  - 备份
  - 恢复
  - 安装
  - 包管理
updated: 2026-06-08 19:36:17
created: 2026-02-26 15:27:36
---

```powershell
#备份
npm list -g --depth=0 > global-packages.txt

#恢复
(gc global-packages.txt | select -skip 1) -replace '^[+|`]-- ' -replace '@[\d.]+$' | ? { $_ -and $_ -ne 'npm' -and $_ -ne 'corepack' } | % { npm i -g $_ }
```
