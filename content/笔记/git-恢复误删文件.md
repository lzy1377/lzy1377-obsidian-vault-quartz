---
title: Git 恢复误删文件
description: 用 `git fsck --lost-found` 找回已删除但 Git 还留有引用的文件
type: troubleshooting
tags:
  - git
  - 恢复
  - 误删
updated: 2026-05-23 10:26:59
created: 2026-04-13 00:35:49
---

# Git 恢复误删文件  
 

---

## 一句话

`git fsck --lost-found` 可以找回已删除但 Git 还留有引用的文件。

## 为什么需要

删除文件或分支后，如果终端还没关（reflog 还在），Git 的悬空对象尚未被垃圾回收，此时可以恢复。常见于手滑删分支没备份的情况。

## 怎么做

```bash
# 1. 查找悬空对象
git fsck --lost-found

# 2. 查看具体内容（替换 <hash>）
git show <hash>

# 3. 恢复到新分支
git checkout -b recover-main <hash>
```

## 踩过的坑

- 执行[Git 清空提交历史](git-清空提交历史.md)中的"删除旧分支"步骤前**必须先备份**，否则删掉的旧分支只能靠 `fsck` 碰运气恢复
- `fsck --lost-found` 只能找到尚未被 `git gc` 清理的对象，时间越久恢复概率越低

## 相关笔记

- [Git 清空提交历史](git-清空提交历史.md) — 清空历史前要备份，就是因为这个教训
- [Git 强制同步](git-强制同步.md) — 另一种可能丢失数据的危险操作
