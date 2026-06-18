---
title: New API 编译安卓版本
description: New API 项目用 NDK + CGO 交叉编译 Android ARM64 的完整命令
type: reference
tags:
  - go
  - android
  - 交叉编译
  - new-api
  - ndk
source: https://docs.newapi.ai/zh/docs/installation/deployment-methods/local-development
updated: 2026-05-23 10:10:47
created: 2026-03-04 06:07:21
---

参考本地开发文档：
https://docs.newapi.ai/zh/docs/installation/deployment-methods/local-development

在项目根目录执行以下命令编译Android ARM64版本：
``` powershell
$NDK="E:\dev\newapi\new-api\build-local\android-ndk-r29"; $BIN="$NDK\toolchains\llvm\prebuilt\windows-x86_64\bin"; $env:CC="$BIN\clang.exe --target=aarch64-linux-android21"; $env:CXX="$BIN\clang++.exe --target=aarch64-linux-android21"; $env:CGO_ENABLED="1"; $env:GOOS="android"; $env:GOARCH="arm64"; $env:CGO_CFLAGS="-fPIC"; $env:CGO_LDFLAGS_ALLOW=".*"; $env:CGO_LDFLAGS="-landroid -llog"; go build -ldflags "-s -w -X 'new-api/common.Version=dev'" -o build-local\new-api-android-arm64
```