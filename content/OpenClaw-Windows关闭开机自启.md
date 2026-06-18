---
title: OpenClaw-Windows关闭开机自启
description: 不能直接删 schtasks，必须重建同名计划任务来替换，否则 OpenClaw 找不到 Gateway
type: troubleshooting
tags:
  - windows
  - schtasks
  - openclaw
  - 开机自启
updated: 2026-06-08 20:18:48
created: 2026-03-27 08:13:14
---

# OpenClaw Windows关闭开机自启
    
	不能直接删掉OpenClaw Gateway 这个schtasks，会导致 openclaw找不到 gateway，只能重建一个名称相同的来替换掉。
```powershell
# 先查看计划任务是否存在
schtasks /Query /TN "OpenClaw Gateway"
schtasks /Query /TN "OpenClaw Gateway" /V /FO LIST

$taskName = "OpenClaw Gateway"  
# 不用windows Terminal只用纯cmd的话，可以不用$wtPath，然后使用下面注释掉的schtasks /Create...
$wtPath = "$env:LOCALAPPDATA\Microsoft\WindowsApps\wt.exe"  
$gateway = "C:\Users\lzy13\.openclaw\gateway.cmd"  
  
schtasks /Delete /F /TN $taskName  

# 注意换行符`后面不能有空格
schtasks /Create /F `
/TN $taskName `
/TR "`"$wtPath`" new-tab cmd /k `"$gateway`"" `
/SC ONCE `
/SD 2099/12/31 `
/ST 23:59 `
/RL HIGHEST

# 不用windows Terminal只用纯cmd的话用这个
# schtasks /Create /F `  
# /TN $taskName `  
# /TR "`"$gateway`"" `  
# /SC ONCE `  
# /SD 2099/12/31 `  
# /ST 23:59 `  
# /RL HIGHEST



# 先查看计划任务是否存在
# schtasks /Query /TN "OpenClaw Gateway"
# 1. 删除计划任务 
# schtasks /Delete /F /TN "OpenClaw Gateway"


#  先查看配置文件夹内容
# dir "%USERPROFILE%\.openclaw" /s
# 2. 删除配置文件夹（这个卸载用） 
# rmdir /s /q "%USERPROFILE%\.openclaw"
```

