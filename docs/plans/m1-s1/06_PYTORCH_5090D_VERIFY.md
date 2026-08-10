# M1-S1 / Step 06 — PyTorch 成功识别 RTX 5090D

> 项目：Flowform / 流形  
> 前置：`05_PYTHON_AI_ENV.md` 已完成  
> 下一步：`07_GPU_TENSOR_SMOKE_TEST.md`

## 目标

在 Flowform 专用 Python 环境中安装并验证适合当前 RTX 5090D 的 PyTorch CUDA 版本，确保 Python 层能够真正创建 CUDA Tensor 并正确识别 GPU。

这是 M1-S1 的关键 Gate 之一。

## 重要约束

PyTorch、CUDA wheel 和 Blackwell / RTX 5090D 支持具有版本时效性。Codex 执行本计划时：

1. **必须优先依据当时官方 PyTorch 安装说明与官方 NVIDIA 兼容信息选择版本。**
2. 不要仅依据仓库文档中写死的旧版本号安装。
3. 不要为了适配某个 PyTorch 包而擅自降级 Windows NVIDIA Driver。
4. 不要修改用户其他 Python / ComfyUI 环境。

## 执行步骤

1. 激活 Flowform 专用 Python 环境。
2. 读取 Step 04 的 GPU / Driver 审计结果。
3. 检查当前官方 PyTorch 对 RTX 5090D / 对应 CUDA runtime 的推荐安装方式。
4. 安装最小 PyTorch 组件：

```text
torch
torchvision（仅在官方组合和后续参考系统需要时）
torchaudio（非必要则暂不安装）
```

避免在本阶段引入大型无关 AI 依赖。
5. 运行基础探测脚本，记录：

```python
import torch

print(torch.__version__)
print(torch.version.cuda)
print(torch.cuda.is_available())
print(torch.cuda.device_count())
print(torch.cuda.get_device_name(0))
print(torch.cuda.get_device_properties(0))
```

6. 验证可以在 `cuda:0` 创建一个最小 Tensor。
7. 记录 PyTorch、CUDA runtime、GPU 名称、device capability、显存等信息。
8. 将最终安装组合写入 Flowform 依赖声明或环境锁定文件。

## 建议新增文件

```text
scripts/verify_pytorch_cuda.py
training/requirements/pytorch.txt
reports/environment/pytorch_cuda.json
reports/environment/pytorch_cuda.md
```

如依赖管理结构已由 Step 05 确定，则遵循已有结构，不重复创建冲突文件。

## 验收输出

必须至少满足：

```text
PyTorch Import             PASS
torch.cuda.is_available    True
CUDA Device Count          >= 1
GPU Name                   RTX 5090D / actual detected name
CUDA Tensor Creation       PASS
PyTorch Version            RECORDED
PyTorch CUDA Runtime       RECORDED
Device Capability          RECORDED
VRAM                       RECORDED
```

## 失败分类

失败时必须判断问题属于哪一层：

- `DRIVER` — 驱动/WSL GPU 不可见；
- `PYTORCH_BUILD` — 安装的 PyTorch build 不支持当前 GPU；
- `CUDA_RUNTIME` — runtime/动态库问题；
- `PYTHON_ENV` — 环境污染或包冲突；
- `UNKNOWN` — 无法确认。

不要用“重装所有 CUDA”作为默认修复策略。

## 不做

- 不跑性能 Benchmark；
- 不训练模型；
- 不安装参考系统；
- 不开始 TensorRT；
- 不修改全局 CUDA 环境；
- 不把一次 `torch.cuda.is_available() == True` 当成完整稳定性测试，稳定性在 Step 07 验证。

## 完成定义

Flowform 专用 Python 环境中的 PyTorch 能稳定识别 RTX 5090D，并成功创建 CUDA Tensor。