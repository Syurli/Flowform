# M1-S1 / Step 07 — GPU Tensor Smoke Test

> 项目：Flowform / 流形  
> 前置：`06_PYTORCH_5090D_VERIFY.md` 已完成  
> 下一步：`08_ENVIRONMENT_REPORT.md`

## 目标

在 Flowform 专用 PyTorch 环境中执行第一套最小 GPU 计算测试，确认 RTX 5090D 不只是“能被识别”，而是真正可以完成 CUDA Tensor 分配、计算、反向传播和同步。

本步骤是环境烟雾测试，不是性能 Benchmark。

## Codex 执行范围

编写一个稳定、可重复执行的测试脚本，至少验证：

1. CUDA Tensor 创建；
2. CPU → GPU 数据传输；
3. 基础矩阵运算；
4. 一个小型神经网络 forward；
5. loss 计算；
6. backward / gradient；
7. optimizer step；
8. CUDA synchronize；
9. 显存分配信息读取；
10. 连续执行多轮无异常。

## 推荐测试结构

测试规模应小到不会对机器造成明显负担，但足以验证真实计算链路。

示意：

```text
Random Input
   ↓
Small MLP
   ↓
Forward
   ↓
Loss
   ↓
Backward
   ↓
Optimizer Step
```

另加一次适中的矩阵乘法或 Tensor 运算，确认基础 CUDA kernel 正常。

## 建议新增文件

```text
scripts/gpu_tensor_smoke_test.py
reports/environment/gpu_tensor_test.json
reports/environment/gpu_tensor_test.md
```

测试脚本应：

- 返回明确 exit code；
- 成功返回 `0`；
- 任一关键测试失败返回非零；
- 捕获并报告 CUDA OOM、unsupported architecture、illegal memory access 等关键异常；
- 不吞掉 traceback。

## 必须记录

```text
PyTorch Version
CUDA Runtime
GPU Name
Device Capability
Test Start / End Time
Forward PASS/FAIL
Backward PASS/FAIL
Optimizer PASS/FAIL
Tensor Math PASS/FAIL
Peak Allocated VRAM
Result
```

性能时间可以记录，但**本阶段不设置性能 KPI**。

## 验收标准

```text
CUDA Allocation        PASS
CPU → GPU Transfer     PASS
Tensor Math            PASS
Forward                PASS
Backward               PASS
Optimizer Step         PASS
CUDA Synchronize       PASS
Repeated Runs          PASS
```

最终总状态：

```text
GPU TENSOR SMOKE TEST: PASS
```

## 不做

- 不做 FP8/量化测试；
- 不做 TensorRT；
- 不进行长时间压力测试；
- 不进行大显存极限测试；
- 不训练动作模型；
- 不用本结果声称 RTX 5090D 的最终 Flowform 推理性能。

## 失败处理

如果失败：

1. 保留完整异常；
2. 标明失败发生在 allocation / forward / backward / sync 哪一步；
3. 回看 Step 04 和 Step 06 报告；
4. 只针对确认的问题修复；
5. 不采用“全部重装”作为第一策略。

## 完成定义

测试脚本可以重复运行，并稳定完成一次真实 PyTorch CUDA forward + backward + optimizer 链路。