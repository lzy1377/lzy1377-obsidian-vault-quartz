---
title: npm 12 生命周期脚本安全策略（allowScripts）
description: npm 12 默认阻止所有依赖包的 install/postinstall 等生命周期脚本执行，需要显式审批才能运行。这是 npm 历史上最大的安全默认策略变更。
type: note
tags:
  - npm
  - 安全
  - 供应链安全
  - node
updated: 2026-08-21 11:11:27
created: 2026-08-21 11:04:48
---
# npm 12 生命周期脚本安全策略（allowScripts）
---
## 一句话

npm 12（2026-07-08 发布）默认阻止所有依赖包的生命周期脚本（preinstall/install/postinstall/node-gyp rebuild）自动执行，必须从"隐式信任"转向"显式审批"。

## 为什么需要

- **供应链攻击入口**：过去任何依赖树中的包都能在 `npm install` 时自动执行代码，是恶意软件分发的主要途径（如 Shai-Hulud 攻击）
- **静默执行风险**：开发者通常不会审查每个间接依赖的 `postinstall` 脚本内容
- **GitHub 定性**：install-time 生命周期脚本是 "npm 生态系统中最大的代码执行面"
- **npm 12 设计哲学**：从"默认允许、手动拒绝"转向"默认拒绝、显式允许"

## 怎么做

| 你的需求 | 命令 | 推荐度 |
| --- | --- | --- |
| 彻底回到 npm 11 行为 | `--dangerously-allow-all-scripts` 或 `.npmrc` | ⭐ 极低，只用于紧急迁移 |
| 一次性批准当前所有包 | `npm approve-scripts --all` | ⭐⭐ 低，图省事但失去审查 |
| 逐个审批已知包 | `npm approve-scripts sharp esbuild ...` | ⭐⭐⭐⭐⭐ 推荐 |
| 查看待审批列表 | `npm approve-scripts --allow-scripts-pending` | ⭐⭐⭐⭐⭐ 推荐 |

### 1. 查看待审批包列表

```bash
npm approve-scripts --allow-scripts-pending
```

### 2. 逐个审批已知需要的包

```bash
# 项目级
npm approve-scripts sharp esbuild bcrypt

# 全局安装时一次性允许
npm install -g opencode-ai --allow-scripts=opencode-ai

# 永久允许某个全局包
npm config set allow-scripts=opencode-ai --location=user
```

### 3. 批量审批当前所有待审批包（折中方案）

```bash
npm approve-scripts --all
```

- 会将所有待审批包按版本锁定写入 `package.json#allowScripts`
- 新加入的包仍需单独审批

### 4. 彻底放开（迁移逃生口，不推荐日常使用）

```bash
# 单次
npm install --dangerously-allow-all-scripts

# 持久化到 ~/.npmrc
npm config set dangerously-allow-all-scripts=true --location=user
```

## 踩过的坑

1. **静默失败**：被阻止的脚本只产生警告，安装仍返回 exit 0，导致用户误以为安装成功
2. **二进制占位符问题**：如 `opencode-ai` 的 `postinstall` 被跳过后，`bin/opencode.exe` 只是一个 479 字节的文本占位符，运行时报错 "not a valid application for this OS platform"
3. **没有通配符支持**：`--allow-scripts=*` 或 `.npmrc` 中 `allow-scripts=all` 均不被支持，npm 故意设计为难以"全允许"
4. **隐式 node-gyp rebuild 也被阻止**：即使包没有显式脚本，只要有 `binding.gyp`，自动编译同样被阻止
5. **全局安装与项目安装策略不同**：CLI 的 `--allow-scripts` 在项目级安装中直接报错，只允许在全局安装中使用

