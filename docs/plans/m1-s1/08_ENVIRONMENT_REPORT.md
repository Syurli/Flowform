# M1-S1 / Step 08 — 生成 EnvironmentReport

> 项目：Flowform / 流形  
> 前置：Steps 02～07 已完成  
> 下一阶段：M1-S2 / ARDY Bring-up

## 目标

把 Windows、WSL2、NVIDIA/CUDA、Python、PyTorch 和 GPU Tensor Test 的结果汇总为一份标准化 `EnvironmentReport`，作为 Flowform M1-S1 的正式交付物和后续所有参考系统实验的环境基线。

## Codex 执行范围

读取前序步骤生成的报告；若某份报告缺失，不要伪造，标记为 `MISSING` 并说明原因。

汇总：

- Windows；
- WSL2 / Ubuntu；
- CPU；
- RAM；
- GPU / VRAM；
- NVIDIA Driver；
- CUDA compatibility / Toolkit（如存在）；
- Python；
- Flowform virtual environment；
- PyTorch；
- PyTorch CUDA Runtime；
- CUDA device capability；
- Git / Git LFS；
- CMake / C++ toolchain；
- GPU Tensor Smoke Test；
- 关键路径；
- WARN / MISSING / BLOCKED 项。

## 标准输出

建议实现：

```text
scripts/build_environment_report.py
reports/environment/EnvironmentReport.json
reports/environment/EnvironmentReport.md
```

同时建立一个可提交到仓库的模板/Schema：

```text
docs/environment/ENVIRONMENT_REPORT_SCHEMA.md
```

如果更适合使用 JSON Schema，可增加：

```text
docs/environment/environment_report.schema.json
```

## EnvironmentReport 建议结构

```text
Flowform Environment Report

Identity
├─ Timestamp
├─ Host
└─ Flowform Commit

Windows
├─ Version
├─ PowerShell
├─ Git / LFS
├─ CMake
└─ C++ Toolchain

WSL2
├─ Distribution
├─ Ubuntu
└─ Kernel

Hardware
├─ CPU
├─ RAM
├─ GPU
└─ VRAM

GPU Software
├─ NVIDIA Driver
├─ CUDA Compatibility
├─ CUDA Toolkit (optional)
└─ WSL GPU Visibility

Python
├─ Version
├─ Executable
├─ Environment
├─ PyTorch
└─ PyTorch CUDA Runtime

Tests
├─ CUDA Available
├─ Tensor Allocation
├─ Forward
├─ Backward
└─ GPU Smoke Test

Overall
└─ READY / READY_WITH_WARNINGS / BLOCKED
```

## 总状态规则

### READY

以下关键项全部通过：

```text
Windows Dev Base       PASS
WSL2                   PASS
GPU Visibility         PASS
Python Environment     PASS
PyTorch CUDA           PASS
GPU Tensor Test        PASS
```

### READY_WITH_WARNINGS

关键链路全部通过，但存在非阻塞项，例如某个可选工具缺失。

### BLOCKED

任一关键链路失败，不能安全进入 ARDY Bring-up。

## 验收标准

1. `EnvironmentReport.json` 可被后续脚本读取。
2. `EnvironmentReport.md` 可直接给人阅读。
3. 报告包含 Flowform 当前 Git commit，确保实验环境可追溯。
4. 不隐藏 warning。
5. 明确给出总状态。
6. 如果状态为 READY 或 READY_WITH_WARNINGS，明确写出：

```text
M1-S1: PASS
Next: M1-S2 ARDY Bring-up
```

## 可选增强

如果实现成本很低，可把前序审计整合为未来的统一入口：

```text
scripts/doctor.ps1
scripts/doctor.sh
```

但本步骤不要求为了追求完整 CLI 而扩大范围。

## 不做

- 不开始安装 ARDY；
- 不开始 MotionBricks / Kimodo；
- 不开始 Flowform Lab；
- 不训练模型；
- 不为了让报告变成 `PASS` 而自动忽略错误。

## 完成定义

产生 Flowform 第一份正式 EnvironmentReport，并能够基于它明确决定是否进入 M1-S2。