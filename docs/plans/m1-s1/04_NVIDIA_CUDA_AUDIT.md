# M1-S1 / Step 04 — NVIDIA / CUDA 状态审计

> 项目：Flowform / 流形  
> 前置：`03_WSL2_AUDIT.md` 已完成  
> 下一步：`05_PYTHON_AI_ENV.md`

## 目标

确认 RTX 5090D 在 Windows 与 WSL2 中都能被正确识别，并记录驱动、CUDA 可见性、显存与基础 GPU 状态，为后续 PyTorch 环境选择提供依据。

本步骤只做 GPU/CUDA 基础能力审计，不把“系统安装了 CUDA Toolkit”与“PyTorch 能使用 CUDA”混为一谈。

## Codex 执行范围

检查并记录：

- Windows NVIDIA Driver 版本；
- GPU 名称；
- VRAM 总量；
- `nvidia-smi` 是否正常；
- Windows 侧 `nvidia-smi` 报告的 CUDA compatibility version；
- 是否存在本机 CUDA Toolkit；若存在，记录 `nvcc` 版本和路径；
- WSL2 内 `nvidia-smi` 是否正常；
- WSL2 是否能看到同一块 GPU；
- GPU 当前基本状态、温度、占用、驱动可见性。

## 执行步骤

1. Windows 运行 `nvidia-smi` 并保存关键字段。
2. 探测 `nvcc` 是否存在；不存在不直接判定失败。
3. 记录 CUDA Toolkit 路径，仅在确实安装时报告。
4. WSL2 内运行 `nvidia-smi`。
5. 对比 Windows 与 WSL2 的 GPU 型号、显存和驱动可见性。
6. 输出机器可读 JSON 与 Markdown 报告。
7. 根据结果给 Step 05 一个明确建议：后续 Python 环境是否可以直接进入 PyTorch CUDA 安装验证。

## 建议新增文件

```text
scripts/audit_gpu.ps1
scripts/audit_gpu.sh
reports/environment/gpu_cuda_audit.json
reports/environment/gpu_cuda_audit.md
```

## 验收标准

至少应明确：

```text
GPU Model             RTX 5090D / RECORDED
VRAM                  ~32 GB / RECORDED
Windows nvidia-smi    PASS
WSL2 nvidia-smi       PASS
Driver Version        RECORDED
CUDA Compatibility    RECORDED
CUDA Toolkit          PRESENT / NOT REQUIRED / WARN
nvcc                  RECORDED IF PRESENT
```

关键 Gate 是 Windows 和 WSL2 都能正常看到 GPU；是否安装独立 CUDA Toolkit 不是本步骤的唯一成功标准。

## 不做

- 不安装 PyTorch；
- 不跑模型训练；
- 不做 TensorRT；
- 不为了匹配某个旧项目随意降级显卡驱动；
- 不在没有必要时安装多套 CUDA Toolkit。

## 停止条件

以下情况停止自动修改并报告：

- Windows `nvidia-smi` 失败；
- WSL2 完全不可见 GPU；
- 修复需要卸载或降级现有显卡驱动；
- 修复可能影响用户现有 ComfyUI / Unreal / CUDA 工作流。

## 完成定义

Windows 与 WSL2 均能稳定识别 GPU，驱动/CUDA 基础状态已经被记录，并足以指导下一步 Python + PyTorch 环境建立。