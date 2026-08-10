# M1-S1 / Step 05 — 建立 Python AI 环境

> 项目：Flowform / 流形  
> 前置：`04_NVIDIA_CUDA_AUDIT.md` 已完成  
> 下一步：`06_PYTORCH_5090D_VERIFY.md`

## 目标

在已确认可用的 WSL2 / Ubuntu 环境中，为 Flowform 建立一套独立、可复现、不会污染系统 Python 的 AI 开发环境。

本步骤只建立 Flowform 自己的 Python 环境与依赖锁定骨架；PyTorch GPU 可用性在下一步单独验收。

## 原则

- Flowform 使用独立虚拟环境，不依赖系统全局 Python 包。
- 不覆盖用户已有 ComfyUI、其他 AI 项目或系统 Python 环境。
- 版本选择应优先满足当前 RTX 5090D + PyTorch 官方可用组合；如需要联网确认版本，先查官方 PyTorch / NVIDIA 文档再固定。
- 不提前安装 ARDY / MotionBricks / Kimodo 的完整依赖。

## Codex 执行范围

1. 检查 WSL2 中当前 Python 版本与包管理能力。
2. 选择 Flowform 的环境管理方式，优先使用简单、可脚本化且便于 CI 复现的方案。
3. 建立项目专用虚拟环境，例如：

```text
workspace/venvs/flowform
```

或等价的 WSL 用户环境位置；不要提交虚拟环境本体。
4. 建立可重复创建环境的脚本。
5. 建立基础依赖声明/锁定骨架。
6. 安装最小必要工具，例如 pip / build / packaging 等基础能力。
7. 验证新 shell 中可以明确激活 Flowform 环境，并确认 `python` / `pip` 都指向该环境。
8. 输出环境元数据。

## 建议新增文件

```text
scripts/setup_python_env.sh
scripts/activate_flowform.sh
training/requirements/base.txt
reports/environment/python_env.json
reports/environment/python_env.md
```

如果实际采用 `pyproject.toml`、`requirements.lock` 或其他更合适的标准结构，可以调整，但必须在文档中说明理由。

## 最小检查

激活后至少验证：

```text
python --version
python -c "import sys; print(sys.executable)"
pip --version
```

并确认 Python 可执行路径属于 Flowform 专用环境。

## 验收标准

```text
Dedicated Python Env     PASS
System Python Untouched  PASS
Python Version           RECORDED
pip                      PASS
Recreate Script          PASS
Activation Script        PASS
Dependency Spec          CREATED
```

要求：删除该虚拟环境后，理论上能够仅依赖仓库脚本重新创建，而不依赖人工记忆。

## 不做

- 不安装三个参考系统；
- 不开始 Flowform 模型代码；
- 不跑训练；
- 不把虚拟环境目录提交 Git；
- 不改系统默认 `python`；
- 不用 `sudo pip install` 污染系统环境。

## 停止条件

- 需要替换系统 Python；
- 依赖安装方案与已有全局环境发生冲突；
- 无法确定 PyTorch/GPU 兼容版本时，不猜版本，记录并等待 Step 06 按官方来源确认。

## 完成定义

Flowform 已拥有独立、可重新创建的 Python AI 环境，并可以安全进入 PyTorch GPU 安装与识别验证。