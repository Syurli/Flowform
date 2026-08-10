# M1-S1 — Workstation Baseline / Codex Execution Index

> 项目：**流形 Flowform**  
> 阶段：M1 — Workstation & Reference Systems Baseline  
> 目标：完成工作站审计、Python/PyTorch GPU 环境和第一份 `EnvironmentReport`。

## 执行方式

这些计划文件设计为**按顺序逐个交给 Codex 执行**。不要一次性把 02～08 全部合并成一个大任务。

每一步执行完成后：

1. 检查该步骤验收标准；
2. 提交该步骤新增/修改的仓库文件；
3. 保留本地 `reports/` 输出；
4. 若步骤状态为 `BLOCKED`，停止后续步骤并先处理阻塞问题；
5. 若为 `PASS` 或允许继续的 `WARN`，再进入下一份计划。

## 顺序

| Step | Plan | 主要输出 | Gate |
|---|---|---|---|
| 02 | [`02_WINDOWS_ENV_AUDIT.md`](02_WINDOWS_ENV_AUDIT.md) | Windows 开发环境审计脚本与报告 | Windows Base |
| 03 | [`03_WSL2_AUDIT.md`](03_WSL2_AUDIT.md) | WSL2 / Ubuntu 审计 | WSL2 Ready |
| 04 | [`04_NVIDIA_CUDA_AUDIT.md`](04_NVIDIA_CUDA_AUDIT.md) | NVIDIA / CUDA 可见性审计 | GPU Visible |
| 05 | [`05_PYTHON_AI_ENV.md`](05_PYTHON_AI_ENV.md) | Flowform 独立 Python 环境 | Python Env |
| 06 | [`06_PYTORCH_5090D_VERIFY.md`](06_PYTORCH_5090D_VERIFY.md) | PyTorch CUDA / RTX 5090D 验证 | PyTorch CUDA |
| 07 | [`07_GPU_TENSOR_SMOKE_TEST.md`](07_GPU_TENSOR_SMOKE_TEST.md) | GPU forward/backward 烟雾测试 | GPU Compute |
| 08 | [`08_ENVIRONMENT_REPORT.md`](08_ENVIRONMENT_REPORT.md) | 标准 EnvironmentReport | **M1-S1 PASS** |

## Codex 通用执行约束

所有步骤共同遵守：

- 优先**检测现有环境**，不要默认重装。
- 不覆盖用户已有 Unreal Engine、ComfyUI、Python、CUDA 或 Visual Studio 环境。
- 不因为版本不一致就擅自降级 NVIDIA Driver。
- 不执行会删除 WSL 数据、覆盖 PATH、替换系统 Python 的破坏性操作。
- 遇到需要重启、BIOS/UEFI 修改、驱动重装、全局工具链替换时，先停止并报告。
- 对具有时效性的 PyTorch / CUDA / RTX 5090D 兼容信息，执行时以**官方 PyTorch / NVIDIA 文档**为准，不依赖计划文件中的静态版本猜测。
- 所有自动化脚本必须给出清晰 exit code 和错误信息。
- 本地环境报告应可追溯到当前 Flowform Git commit。
- `reports/`、虚拟环境、缓存和大型下载内容不得提交 Git。

## M1-S1 最终成功标准

```text
Windows Dev Base       PASS
WSL2                   PASS
GPU / Driver           PASS
Python Environment     PASS
PyTorch CUDA           PASS
GPU Tensor Smoke Test  PASS
EnvironmentReport      PASS
```

最终输出：

```text
M1-S1: PASS
Next: M1-S2 ARDY Bring-up
```

完成后再进入 ARDY，不要提前并行安装 ARDY / MotionBricks / Kimodo。