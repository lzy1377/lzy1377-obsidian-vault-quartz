---
title: Git 强制同步
description: 两种方向：`push --force` 用本地覆盖远程，`reset --hard` 用远程覆盖本地
type: how-to
tags:
  - git
  - force-push
  - reset
  - 同步
updated: 2026-05-23 10:09:24
created: 2026-04-13 00:35:49
---

# Git 强制同步

## 一句话

两种强制同步方向：`push --force` 用本地覆盖远程，`reset --hard` 用远程覆盖本地。

## 为什么需要

- 本地重写了历史（如 rebase、清空 commit），需要覆盖远程
- 本地改乱了，想直接丢弃所有修改以远程为准

## 怎么做

### 方向一：本地覆盖远程

**场景**：本地有未推送的提交，确定要覆盖远程历史。

```bash
# 强制推送（直接覆盖）
git push --force origin <分支名>

# 更安全的强制推送（远程被他人修改时拒绝）
git push --force-with-lease origin <分支名>
```

### 方向二：远程覆盖本地

**场景**：本地有未提交的修改，确定以远程为准。

```bash
git fetch origin
git reset --hard origin/<分支名>
```

本地所有未提交的修改会被丢弃，与远程完全一致。

## 踩过的坑

| 命令 | 风险 |
|------|------|
| `push --force` | 直接覆盖远程历史，协作者的本地分支会与远程产生冲突 |
| `push --force-with-lease` | 更安全但不绝对——只保护"远程没被改过"的情况 |
| `reset --hard` | **不可逆**，本地未提交的修改直接消失 |

- 优先用 `--force-with-lease` 而非 `--force`
- `reset --hard` 前用 `git stash` 暂存一下，给自己留退路
- 如果 `reset --hard` 后发现丢错了东西，参考 [Git 恢复误删文件](git-恢复误删文件.md)

## 相关笔记

- [Git 清空提交历史](git-清空提交历史.md) — 清空历史后通常需要 force push
- [Git 恢复误删文件](git-恢复误删文件.md) — `reset --hard` 后的补救措施
