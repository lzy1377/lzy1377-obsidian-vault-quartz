---
title: Go 环境配置与交叉编译
description: Go 环境变量配置、全局常用包安装、Windows 下交叉编译 Android ARM64
type: how-to
tags:
  - go
  - windows
  - 环境配置
  - 交叉编译
  - 包管理
updated: 2026-05-23 10:10:41
created: 2026-04-13 02:29:09
---

## 1. `GOROOT` 和 `GOPATH`
在 Windows 上设置 `GOROOT` 和 `GOPATH` 环境变量
 1. 概念说明
- **GOROOT**: Go 语言的安装目录（包含编译器、标准库等）。**通常不需要手动设置**，Go 安装程序会自动配置。
- **GOPATH**: 你的全局工作区目录，存放第三方包和项目代码。默认使用 `%USERPROFILE%\go`（如 `C:\Users\用户名\go`）。

## 2. Go 全局常用包安装
```powershell
# 基础四件套
go install golang.org/x/tools/gopls@latest                    # 语言服务器
go install github.com/go-delve/delve/cmd/dlv@latest           # 调试器
go install honnef.co/go/tools/cmd/staticcheck@latest          # 静态检查
go install mvdan.cc/gofumpt@latest                            # 格式化
go install golang.org/x/tools/cmd/goimports@latest            # 自动导入

# CI/质量保障
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
go install golang.org/x/vuln/cmd/govulncheck@latest

```

## 3. Go 跨平台编译安卓
	以windows编译安卓arm64为例
	先下载ndk：https://developer.android.google.cn/ndk/downloads?hl=zh-cn
```powershell
$NDK="path to\android-ndk"; $BIN="$NDK\toolchains\llvm\prebuilt\windows-x86_64\bin"; $env:CC="$BIN\clang.exe --target=aarch64-linux-android21"; $env:CXX="$BIN\clang++.exe --target=aarch64-linux-android21"; $env:CGO_ENABLED="1"; $env:GOOS="android"; $env:GOARCH="arm64"; $env:CGO_CFLAGS="-fPIC"; $env:CGO_LDFLAGS_ALLOW=".*"; $env:CGO_LDFLAGS="-landroid -llog"; go build -ldflags "-s -w -X 'xxx'" -o xxx-android-arm64
```
-ldflags "-s -w -X 'xxx'"是Go 编译器的**链接器标志（linker flags）**，用于控制编译过程和优化输出文件。这三个标志分别用于**缩减体积**和**注入版本信息**：
### 各标志含义
1. `-s` —— 剥离符号表
- **作用**：删除符号表和调试信息
- **效果**：显著减小二进制文件体积（通常可减少 20-30%）
- **副作用**：无法使用 `gdb`、`delve` 等调试器进行调试
2. `-w` —— 剥离 DWARF 信息
- **作用**：删除 DWARF 调试信息（一种标准的调试数据格式）
- **效果**：进一步减小体积
- **与 -s 的关系**：`-s` 包含 `-w` 的功能，但两者一起使用是常见做法（确保最大程度优化）
3. `-X 'new-api/common.Version=dev'` —— 编译时注入变量
- **作用**：在链接阶段修改指定包中的**字符串变量**值
- **格式**：`-X importpath.name=value`

