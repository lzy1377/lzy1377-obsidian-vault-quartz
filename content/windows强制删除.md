---
title: windows强制删除
description: windows强制删除指定目录
type: note
tags:
  - windows
  - 删除
  - 卸载
updated: 2026-08-31 13:53:31
created: 2026-07-01 09:25:22
---

```powershell
Remove-Item -Path "删除目录" -Recurse -Force -ErrorAction SilentlyContinue
```