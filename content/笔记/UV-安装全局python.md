---
title: UV 安装全局 Python 环境
description: 创建固定位置 venv 并置顶系统 PATH，让 Agent 的 `python` 命令自动命中虚拟环境
type: how-to
tags:
  - python
  - uv
  - venv
  - windows
  - agent
  - PATH
  - 安装
updated: 2026-06-08 19:36:21
created: 2026-05-22 23:34:07
---

# UV 伪全局虚拟环境：解决全局 python 和依赖问题 (Windows)

> uv 禁止向全局 Python 装包。创建一个固定位置的 venv，将其 `Scripts` 目录置顶到 **系统 PATH**（高于用户 PATH），使其成为 `python` 命令的默认解析目标。Agent 执行 `python xxx.py` 时自动命中该环境，无需激活。

---

## 问题

- `uv python install` 安装的 Python 被标记为 `externally managed`（PEP 668），禁止 `uv pip install --system`。
- 例如Agent/Skill 自动调用 `python xxx.py`，硬编码依赖全局 Python 环境。
- 需要一种方案，既遵守 uv 的隔离哲学，又满足 Agent 的全局调用需求。

---

## 为什么不用其他方案

- **再装官方 Python**：回到 `pip install` 污染全局的老路，与 uv 的多版本管理冲突。
- **`uv run --with`**：Agent 调用链是 `python` 而非 `uv run`，无法直接适配。
- **删除 `EXTERNALLY-MANAGED` 文件**：破坏 uv 的设计约束，后续升级会被覆盖。

---

## 方案：伪全局 Venv + PATH 置顶

### 1. 创建固定虚拟环境

```powershell
# --seed可以安装pip，这样venv自带pip
uv venv --seed --python <python_version> <path_to_venv>
```

### 2. 置顶 PATH（关键步骤）

**推荐做法：使用环境变量中转**  
先创建系统级变量 `VENV_GLOBAL`，再在 PATH 中引用 `%VENV_GLOBAL%\Scripts`。好处是：
- 以后迁移 venv 位置时，**只需改 `VENV_GLOBAL` 一个变量**，无需编辑冗长的 PATH 字符串。
- 避免 PATH 中硬编码绝对路径，降低误删和截断风险。

#### 方法一：PowerShell 命令（管理员权限）

```powershell
# ========== 步骤 A：创建系统环境变量 VENV_GLOBAL ==========
# 作用：指向你的伪全局 venv 根目录。以后换位置只需改这里。
$venvRoot = "C:\Users\<user>\.skill-venv"
[Environment]::SetEnvironmentVariable("VENV_GLOBAL", $venvRoot, "Machine")

# ========== 步骤 B：将 %VENV_GLOBAL%\Scripts 置顶到系统 PATH ==========
# 作用：Windows PATH 从左到右搜索，置顶可确保 `python` / `pip` 命令最先命中我们的 venv。
#       使用 %VENV_GLOBAL% 变量引用，而非写死路径。
$sysPath = [Environment]::GetEnvironmentVariable("PATH", "Machine")
$newSysPath = "%VENV_GLOBAL%\Scripts;" + $sysPath
[Environment]::SetEnvironmentVariable("PATH", $newSysPath, "Machine")
````

> **注意**：
> - 必须以**管理员身份**运行 PowerShell，否则无法写入系统环境变量。
> - 修改后必须**重启终端/IDE**（甚至重启电脑），新进程才会继承更新后的系统 PATH。
> - `%VENV_GLOBAL%` 是 Windows 的变量引用语法；在 PowerShell 中写入时**保留原样字符串**，Windows 会在运行时展开它。

#### 方法二：图形化界面操作

适合不熟悉命令行或需要可视化确认每一步的用户。

1. **打开环境变量编辑界面**
   - 按 `Win + R`，输入 `sysdm.cpl`，回车。
   - 切换到 **"高级"** 选项卡 → 点击底部 **"环境变量(N)..."** 按钮。

1. **创建系统变量 `VENV_GLOBAL` **
   - 在下半部分 **"系统变量(S)"** 区域点击 **"新建(W)..."**。
   - 变量名：`VENV_GLOBAL`
   - 变量值：<path_to_venv>（你的 venv 根目录，不含 `\Scripts`）
   - 点击 **"确定"**。

3. **编辑系统 PATH 并置顶**
   - 在 **"系统变量"** 列表中找到 `Path` → 双击或点击 **"编辑(I)..."**。
   - **添加新条目并置顶**：
     - 点击 **"新建(N)"** → 输入 `%VENV_GLOBAL%\Scripts`
     - 选中这条，点击右侧 **"上移(U)"** 按钮，一直移到**最顶部**。
   - 点击 **"确定"** 关闭 PATH 编辑窗口。

4. **保存并生效**
   - 在环境变量主窗口点击 **"确定"**。
   - 在系统属性窗口点击 **"确定"**。
   - **重启所有终端和 IDE**（PowerShell、VS Code、Cursor 等），必要时重启电脑。

### 3. 验证（必须新开终端）

```powershell
Get-Command python
# 期望: C:\Users\<user>\.skill-venv\Scripts\python.exe

python -c "import requests; print(requests.__version__)"
```

---

## 核心原理

- **免激活**：虚拟环境的 `python.exe` 可直接调用。`activate` 只是给人类简化 PATH 的辅助脚本；Agent 机器调用不依赖激活状态。
- **PATH 优先级**：Windows 拼接 PATH 为 `系统;用户` （先系统后用户），从左到右搜索。置顶系统 PATH 可确保 `python` 命令被劫持到 `<path_to_venv>`。

---

## 维护

```powershell
# 增删包（无需激活）
uv pip install <package>
# 上面无效果，执行下一行命令强制指定python
<path_to_venv>\Scripts\python.exe -m uv pip install <package>

# 若当前终端已继承置顶 PATH，可直接用
uv pip list
```
- 如果 venv 里面有 pip，直接使用 pip install 也行
```powershell
# 增删包（无需激活）
pip install <package>
# 若当前终端已继承置顶 PATH，可直接用
pip list
```
---

## 故障排查

| 现象 | 原因 | 解决 |
|------|------|------|
| `Get-Command python` 仍指向旧路径 | 系统 PATH 残留旧 Python | 清理系统 PATH 中的 `C:\Python...` 和 `WindowsApps` 条目 |
| Agent 硬编码了 python 绝对路径 | Agent 配置绕过 PATH | 修改 Agent 配置中的解释器路径为 `.skill-venv\Scripts\python.exe` |
| 包安装后仍报 `ModuleNotFoundError` | Agent 用了不同的 Python 进程 | 检查 Agent 日志中的实际调用路径 |
