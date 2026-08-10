# M1-S1 / Step 02 — Windows 开发环境审计

> 项目：Flowform / 流形  
> 阶段：M1 — Workstation & Reference Systems Baseline  
> 前置：主仓库已建立  
> 下一步：`03_WSL2_AUDIT.md`

## 目标

确认当前 Windows 11 主机是否具备 Flowform 后续开发所需的基础工具，并把结果写成可复查的机器状态记录。

本步骤**以检查为主，不主动升级或替换已有工具链**。发现缺失项时记录，只有明确、安全且不会破坏现有 UE/C++ 开发环境的组件才可补装。

## Codex 执行范围

检查并记录：

- Windows 版本、Build、架构；
- PowerShell 版本；
- Git；
- Git LFS；
- VS Code（如已安装）；
- Visual Studio / Build Tools 及可用 C++ Toolchain；
- CMake；
- Ninja（如已有）；
- 可用磁盘空间；
- 系统内存；
- 当前 PATH 中与 Python / CUDA / CMake / Git 有关的关键路径。

## 执行步骤

1. 读取系统信息，不修改系统配置。
2. 逐项探测上述工具是否存在，并记录版本与实际可执行路径。
3. 检查 `git lfs env` 是否可正常执行。
4. 检查至少一个 MSVC C++ 编译工具链是否存在；不要因未找到某一特定版本就覆盖安装已有 Visual Studio。
5. 检查仓库所在磁盘与建议的 `workspace/` 所在磁盘剩余空间。
6. 输出机器可读 JSON 与人类可读 Markdown 两份审计结果。
7. 如有缺失项，在报告中标记 `MISSING` 或 `WARN`，不要隐藏。

## 建议新增/修改文件

Codex 可实现：

```text
scripts/audit_windows.ps1
reports/environment/windows_audit.json
reports/environment/windows_audit.md
```

`reports/` 默认被 `.gitignore` 忽略，因此实际执行报告可保留在本地；脚本应提交到仓库。

## 状态定义

- `PASS`：可直接用于后续步骤。
- `WARN`：存在非阻塞问题。
- `MISSING`：缺少后续明确需要的组件。
- `BLOCKED`：无法继续下一步。

## 验收标准

必须能够明确回答：

```text
Windows              PASS/WARN
PowerShell           PASS/WARN
Git                  PASS/MISSING
Git LFS              PASS/MISSING
C++ Toolchain        PASS/MISSING
CMake                PASS/MISSING
Disk Space           PASS/WARN
RAM                   PASS/WARN
```

并生成一份包含版本号和实际路径的审计报告。

## 停止条件

遇到以下情况时停止自动修改，只报告问题：

- 需要卸载或替换已有 Visual Studio；
- 需要修改全局 CUDA / Python 环境以修复冲突；
- 需要重装显卡驱动；
- 需要删除用户现有 PATH 项；
- 发现会影响现有 Unreal Engine/C++ 项目的明显风险。

## 完成定义

当 Windows 基础环境已经可被脚本重复审计，并且没有阻止 WSL2 检查的阻塞项时，本步骤完成。