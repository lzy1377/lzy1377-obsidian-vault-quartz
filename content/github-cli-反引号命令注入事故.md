---
title: GitHub CLI 反引号命令注入事故
description: 用 gh issue create --body 提交 Markdown 反引号内容时，shell 误把反引号里的 git 命令执行了，导致意外 force push
type: troubleshooting
tags: [github-cli, shell, git, security, 事故]
updated: 2026-06-19 04:21:15
created: 2026-06-19 04:21:15
---

# GitHub CLI 反引号命令注入事故

## 一句话

用 `gh issue create --body "..."` 提交含 Markdown 反引号的内容时，shell 把反引号里的命令当成命令替换执行了，结果意外执行了 `git reset --hard` 和 `git push --force`。

## 为什么需要

写 issue 正文时经常会用反引号包命令，比如 `` `git reset --hard HEAD~1` ``。如果直接传给 `gh --body "..."`，shell 会先把反引号里的内容执行一遍，再把执行结果塞回字符串。这会导致：

- issue 正文内容被污染（命令输出替换了原本的文字）
- 更严重的是，如果反引号里是 `git reset --hard` 或 `git push --force`，会真的修改仓库历史

## 怎么做

### 安全写法 1：用 --body-file

把正文写进文件，再传给 `gh`：

```bash
cat > issue-body.md << 'EOF'
## 问题描述

通过 `git reset --hard` + `git push --force` 回退...
EOF

gh issue create --title "标题" --body-file issue-body.md
```

### 安全写法 2：用单引号 heredoc

```bash
gh issue create --title "标题" --body "$(cat << 'EOF'
## 问题描述

通过 `git reset --hard` 回退...
EOF
)"
```

### 避免写法

```bash
# 危险！反引号会被 shell 执行
gh issue create --body "执行 `git reset --hard HEAD~1` 后..."
```

## 踩过的坑

- 本次事故中，issue 正文里的 `` `git reset --hard HEAD~1` `` 被 shell 执行，导致本地 `main` 被重置到更早的提交，并 force push 到了 GitHub/Gitee，删除了一个已经存在的 commit。
- 即使字符串用双引号 `""` 包裹，反引号仍然会被 shell 解析；只有用单引号 heredoc 或文件方式才能避免。
- `gh issue create` 本身没问题，问题在于 shell 的命令替换语法。
- 修复时只能依赖 `git reflog` 找到被删的 commit，再 `git reset --hard` 回去并再次 force push 恢复。

## 相关笔记

- [Git 强制同步](git-强制同步.md)
- [Git 恢复误删文件](git-恢复误删文件.md)
