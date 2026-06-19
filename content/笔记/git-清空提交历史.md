---
title: Git 清空提交历史
description: 用 `--orphan` 创建无父提交的分支，保留所有文件内容但彻底清空 commit 历史
type: how-to
tags:
  - git
  - 历史重写
  - orphan
updated: 2026-05-23 10:09:30
created: 2026-04-13 00:35:49
---

# Git 清空提交历史

## 一句话

用 `--orphan` 创建无父提交的分支，保留所有文件内容但彻底清空 commit 历史。

## 为什么需要

- 仓库历史过于臃肿，想从零开始
- 提交历史中包含敏感信息需要彻底清除
- 把实验性仓库转成正式仓库

## 怎么做

### 分步执行

```bash
# 1. 备份（必须！）
git branch backup-before-clean

# 2. 创建孤儿分支
git checkout --orphan clean-main

# 3. 清空暂存区
git rm -rf .

# 4. 从备份恢复文件
git checkout backup-before-clean -- .

# 5. 提交所有文件
git add -A
git commit -m "Initial commit"

# 6. 替换旧分支
git branch -D main
git branch -m main
```

### 一键执行

```bash
git branch backup-before-clean && \
git checkout --orphan clean-main && \
git rm -rf . && \
git checkout backup-before-clean -- . && \
git add -A && \
git commit -m "Initial commit" && \
git branch -D main && \
git branch -m main
```

### 验证结果

```bash
git log --oneline   # 应只有一条 commit
git status          # 确认文件都在
```

## 关键点说明

| 步骤 | 作用 |
|------|------|
| `--orphan` | 创建无父提交的新分支，彻底断开历史 |
| `git rm -rf .` | 清空暂存区，防止旧文件残留 |
| `checkout -- .` | 从备份恢复所有文件内容 |
| `git add -A` | 添加所有文件（含未跟踪的） |

## 踩过的坑

- **此操作不可逆**，所有历史记录永久删除。务必先建备份分支
- 如果已推送到远程，需要 `git push --force` 覆盖远程
- 确认无误后可删除备份分支 `backup-before-clean`
- 不建备份就直接执行的话，删掉的旧分支只能碰运气用 [Git 恢复误删文件](git-恢复误删文件.md) 恢复

## 相关笔记

- [Git 恢复误删文件](git-恢复误删文件.md) — 没备份就执行此操作的补救措施
- [Git 强制同步](git-强制同步.md) — 覆盖远程时的 `--force` 用法
