# M1-S3 — Control Operators Python Reference

> 目标：跑通公开的简化 Control Operators Reference，实现从 Dataset 到交互式 Autoregressive Flow Controller 的透明 Baseline。

## 重要边界

公开 Python Reference 用于解释论文核心方法，但不是 Epic AnimGen / 论文原始生产实现的完整源码替代品。

如上游许可证未明确：

- 不复制代码到 Flowform 核心；
- 在 `third_party/` 或外部 workspace 中运行；
- 只提交环境脚本、实验摘要和独立实现结论；
- 保留 Source URL / Commit。

## Step 1｜Source / Environment

记录：

```text
Repository URL
Commit SHA
Python
PyTorch
CUDA
GPU
Dataset
Install Commands
```

Gate：`CONTROL_OP_SOURCE_READY=PASS`

## Step 2｜Dataset / Motion Representation

确认：

- 使用的数据；
- Skeleton；
- Pose feature；
- Velocity feature；
- Normalization；
- Window / sample construction。

输出：`Architecture Evidence Card`。

## Step 3｜Control Operators

逐个记录 Reference 中实际存在的 Operator / Schema 组合方式。

回答：

```text
一个 Control 如何定义？
多个 Control 如何组合？
如何编码到网络输入？
Schema 与 Runtime Control Object 如何对应？
```

不要用 AnimGen API 中存在但 Python Reference 没有实现的功能填补文档。

## Step 4｜Training Baseline

先跑官方 / README 推荐的最小训练路径。

记录：

- config；
- iterations / epochs；
- duration；
- peak VRAM；
- loss；
- checkpoint；
- output size。

如果全量训练很长，可以先做 Smoke Test，但必须明确：

```text
Training Smoke = PASS
Full Baseline = DEFERRED
```

不能把 smoke test 写成完整训练完成。

## Step 5｜Interactive Runtime

运行交互式 Controller，并复用：

```text
A01 Idle
A02 Idle -> Walk
A03 Idle -> Run
A04 Run -> Stop
A05 Stop -> Recover
A06 90 Turn
A07 180 Turn
A08 Speed Change
A09 Rapid Direction Change
```

重点观察：

- frame-by-frame generation；
- previous pose feedback；
- control change response；
- drift；
- foot slide；
- stochastic variation（若存在）；
- runtime cost。

## Step 6｜与 AnimGen 对照

只比较能确认的结构：

| Dimension | Python Reference | AnimGen Player |
|---|---|---|
| Pose Encoding | | |
| Control Schema | | |
| Controller Model | | |
| Flow Steps | | |
| Training UX | CLI / code | Editor |
| Runtime | Python | UE |
| LOD / Distillation | | |
| Reproducibility | | |

不要因为名称相似就假设参数或内部实现完全一致。

## Step 7｜Flowform Decision Notes

回答：

1. Flowform V0-A 是否值得采用 Latent Autoregressive Flow Matching？
2. AutoEncoder 的收益和成本？
3. Control Schema V0 最少需要哪些类型？
4. 一个最小 Flowform 独立实现需要哪些模块？
5. 哪些逻辑属于通用 Framework，哪些只是 Reference Model 实现？

## Final Gate

```text
Source / Environment       PASS
Dataset                    PASS
Control Schema             PASS
Training Smoke/Baseline    PASS
Interactive Runtime        PASS
AnimGen Comparison         PASS
Architecture Notes         PASS

M1-S3 = PASS
Next = M1-S4 ARDY Control / Replan Baseline
```
