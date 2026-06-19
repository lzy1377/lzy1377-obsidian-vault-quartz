---
title: Redis Windows 安装
description: Redis Windows 安装并注册服务
type: note
tags:
  - Redis
  - windows
  - 安装
  - 开机自启
updated: 2026-06-08 18:31:21
created: 2026-06-05 11:28:17
---

# Redis Windows 服务安装笔记
---
https://github.com/redis-windows/redis-windows/
## 环境

- **版本**: Redis 8.8.0
- **平台**: Windows 11 x 64 (MSYS 2 + Cygwin 运行时)
- **路径**: `D:\Devtools\Redis\Redis-8.8.0-Windows-x64-msys2-with-Service`

## 配置修改

基于 `redis.conf` 做了以下修改（保留所有注释，仅改配置行）：

| 配置项 | 原值 | 新值 | 说明 |
|--------|------|------|------|
| `logfile` | `""` | `"./logs/redis.log"` | 日志写入文件 |
| `requirepass` | `# requirepass foobared` | `requirepass root` | 密码设为 `root` |
| `maxmemory` | `# maxmemory <bytes>` | `maxmemory 2gb` | 最大内存 2 GB |
| `appendonly` | `no` | `yes` | 开启 AOF 持久化 |

其余关键默认配置保持不变：

- `bind 127.0.0.1 -::1` — 仅监听本地
- `protected-mode yes` — 保护模式开启
- `port 6379` — 标准端口
- `dir ./` — 工作目录为当前目录
- `save 3600 1 300 100 60 10000` — RDB 快照（默认策略）
- `appendfsync everysec` — AOF 每秒刷盘

## 注册 Windows 服务

以**管理员身份**运行 PowerShell：

```powershell
sc.exe create "Redis" binpath="D:\Devtools\Redis\Redis-8.8.0-Windows-x64-msys2-with-Service\RedisService.exe -c D:\Devtools\Redis\Redis-8.8.0-Windows-x64-msys2-with-Service\redis.conf" start= AUTO
```

启动服务：

```powershell
net start Redis
```

## 卸载服务

```powershell
sc.exe stop Redis
sc.exe delete Redis
```

## 连接 Redis

```powershell
redis-cli
AUTH root
```

## 注意事项

- `start= AUTO` 中 `=` 后面的空格不能省略（`sc.exe` 语法要求）
- 路径含空格时不需要额外加引号，`sc.exe` 的 `binpath=` 会自动处理
- 数据文件和日志分别在 `data/` 和 `logs/` 目录下
- 此为本地开发环境配置，生产环境请用 Linux + 强密码
