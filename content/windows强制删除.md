---
title: windows强制删除
description:
type: note
tags:
  - windows
  - 删除
  - 卸载
updated: 2026-07-03 15:26:11
created: 2026-07-01 09:25:22
---

```powershell
Remove-Item -Path "删除目录" -Recurse -Force -ErrorAction SilentlyContinue
```