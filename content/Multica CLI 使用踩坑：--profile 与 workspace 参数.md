---
title: Multica CLI 使用踩坑：--profile 与 workspace 参数
description: 桌面端自带的 multica CLI 在手动运行时需要带 profile 与 workspace 参数，以及 agent 权限边界，避免踩坑
type: note
tags:
  - multica
  - cli
  - mcp
  - 踩坑
updated: 2026-08-24 14:38:19
created: 2026-08-24 14:35:29
---

# Multica CLI 使用踩坑：--profile 与 workspace 参数

---

## 一句话

手动运行桌面端自带的 `multica` CLI 时，命令要带 `--profile desktop-api.multica.ai` 复用桌面端已登录凭据，workspace 级命令还要补上第二个位置参数（workspace slug），否则会踩两个坑。

## 为什么需要

桌面端安装后自带一个 `multica` CLI（位于 `C:\Users\lzy13\AppData\Local\Programs\@multicadesktop\resources\...\bin\multica.exe`），加入 PATH 后可以手动执行命令操作 workspace。但它的认证配置按 **profile 隔离**：

- 桌面端 daemon 用的独立 profile：`desktop-api.multica.ai`，配置在 `C:\Users\lzy13\.multica\profiles\desktop-api.multica.ai\config.json`，里面已经有 server_url 和登录 token，桌面端一直在用。
- 手动运行 CLI 走的是**默认 profile（default）**，从未配置过。

所以不带参数直接跑，会碰到「没登录」和「不知道在哪个 workspace」两个问题。确认这两个参数，才能在命令行顺手完成 MCP 配置等操作。

## 怎么做

给桌面环境手动运行 multica 的两条铁律：

1. 手动命令统一加 `--profile desktop-api.multica.ai`（复用桌面端已登录凭据，无需重新授权）。
2. workspace 级命令（如 `workspace mcp ...`、`agent mcp ...` 里涉及 workspace 的）必须带上**第二个位置参数** workspace 的 id / slug / prefix。当前 slug：`lzy1377-workspace`。

```powershell
# 加 MCP server 到 workspace 共享库
multica --profile desktop-api.multica.ai workspace mcp add hindsight lzy1377-workspace --server-config '{"type":"http","url":"http://127.0.0.1:34560/mcp/"}'
multica --profile desktop-api.multica.ai workspace mcp add cognee    lzy1377-workspace --server-config '{"type":"http","url":"http://127.0.0.1:34570/mcp"}'

# 查看 / 分配
multica --profile desktop-api.multica.ai workspace mcp list lzy1377-workspace
multica --profile desktop-api.multica.ai agent mcp add <agent-id> <server-id>
```

如果不想每条都带 `--profile`，也可以先执行一次 `multica login` 给默认 profile 单独授权（这只写 default profile，不影响桌面端）。推荐方案：直接用 `--profile desktop-api.multica.ai`，最快最省事。

## 踩过的坑

1. ** `No server configured. Run 'multica setup' first.` ** — 原因：手动 CLI 走的是未配置的默认 profile，不是桌面端出问题。解决：加 `--profile desktop-api.multica.ai`，或 `multica login`。
2. ** `workspace ID is required: pass an id/slug/prefix as argument or set MULTICA_WORKSPACE_ID` ** — 原因：workspace 级命令缺少第二个位置参数。解决：命令里补上 slug `lzy1377-workspace`。
3. **agent 身份改 MCP 配置会被拒（403）** — `workspace mcp add` 报 `agents cannot modify the workspace MCP servers`，`agent mcp add` 报 `agents cannot modify MCP server assignments`。这两类写操作必须由 workspace owner（人类账号）执行，agent 只能 list 读取。
