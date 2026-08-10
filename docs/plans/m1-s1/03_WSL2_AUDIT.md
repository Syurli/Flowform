# M1-S1 / Step 03 — WSL2 环境审计

> 项目：Flowform / 流形  
> 前置：`02_WINDOWS_ENV_AUDIT.md` 已完成  
> 下一步：`04_NVIDIA_CUDA_AUDIT.md`

## 目标

确认 WSL2 与 Ubuntu 环境可稳定承担 Flowform 的 AI / Training 工作负载，并建立可重复检查脚本。

本步骤优先验证现状，不在没有必要时重建现有 WSL 发行版。

## Codex 执行范围

检查：

- WSL 功能状态与版本；
- 默认 WSL 版本是否为 2；
- 已安装发行版及其 WSL 版本；
- Ubuntu 版本；
- Linux kernel；
- systemd 状态；
- WSL 内 Git、CMake、编译器等基础工具；
- Windows 仓库路径与 WSL 文件系统访问是否正常；
- Windows ↔ WSL 基础命令调用是否正常；
- WSL 内磁盘空间与内存可见情况。

## 执行步骤

1. 在 Windows 侧运行 WSL 状态检查。
2. 枚举当前安装的发行版，不删除、不重置已有发行版。
3. 确认用于 Flowform 的 Ubuntu 发行版运行在 WSL2。
4. 进入该发行版，记录 Ubuntu、kernel、shell 和关键基础工具版本。
5. 验证从 WSL 可读取 Flowform 仓库。
6. 验证 Windows 与 WSL 间的基础文件访问和命令调用。
7. 输出 JSON + Markdown 审计结果。
8. 如 WSL2/Ubuntu 缺失，可提供安装脚本或明确命令，但遇到需要重启、BIOS 虚拟化设置或会影响现有发行版的数据操作时停止并报告。

## 建议新增文件

```text
scripts/audit_wsl.ps1
scripts/audit_wsl.sh
reports/environment/wsl_audit.json
reports/environment/wsl_audit.md
```

## 验收标准

最终状态至少应为：

```text
WSL Installed          PASS
WSL Version            2
Ubuntu                 PASS
Ubuntu Version         RECORDED
Linux Kernel           RECORDED
Repo Access            PASS
Windows ↔ WSL IO       PASS
Disk / Memory          RECORDED
```

并明确哪一个发行版是 Flowform 的默认训练环境。

## 不做

- 不安装 PyTorch；
- 不判断 PyTorch CUDA 是否可用；
- 不开始 ARDY / MotionBricks / Kimodo；
- 不重置或删除用户已有 WSL 数据；
- 不为了“干净”而创建多套重复 Ubuntu 环境。

## 停止条件

以下情况停止并报告：

- WSL2 依赖 BIOS/UEFI 虚拟化但当前未启用；
- 操作需要重启系统；
- 现有 Ubuntu 中存在重要环境且修复方案会覆盖数据；
- 当前发行版不是 WSL2，转换存在明显数据风险。

## 完成定义

存在一个确定的 Ubuntu/WSL2 环境，可由脚本重复检查并稳定访问 Flowform 仓库。