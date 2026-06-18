---
title: Git 移除已追踪文件
description: .gitignore 只对未追踪文件生效，已被追踪的文件需用 `git rm --cached` 从索引移除
type: troubleshooting
tags:
  - git
  - gitignore
  - rm-cached
updated: 2026-05-23 10:09:36
created: 2026-04-13 00:35:49
---

# Git 移除已追踪文件（.gitignore 失效）

## 一句话

`.gitignore` 只对未追踪文件生效。已被追踪的文件需要先用 `git rm --cached` 从索引中移除。

## 为什么需要

把文件加入 `.gitignore` 后，Git 依然追踪它的变更——因为该文件在加入忽略规则之前已经被 `git add` 过。常见于项目中期补加 `.gitignore` 规则的场景。

## 怎么做

```bash
# 1. 确认文件当前被追踪
git ls-files | Select-String -Pattern "文件名"

# 2. 从索引移除（保留本地文件）
git rm --cached 文件路径

# 3. 提交并推送
git commit -m "Remove 文件路径 from Git tracking (should be ignored)"
git pull --rebase
git push
```

## 踩过的坑

- `git rm`（不带 `--cached`）会**同时删除本地文件**，务必加 `--cached`
- 提交后其他协作者 pull 时，该文件会从他们的工作区消失——需提前通知团队备份

## 相关笔记

- [Git 清空提交历史](git-清空提交历史.md) — 如果想一劳永逸清理整个仓库的追踪历史
- [Git 强制同步](git-强制同步.md) — push 时如果冲突，可能需要 force
