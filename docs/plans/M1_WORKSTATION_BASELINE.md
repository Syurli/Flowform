# M1｜工作站与参考系统基线

> 项目：**流形 Flowform**  
> 阶段周期：**2026-08-10 ～ 2026-09-09**  
> 本次校准：**2026-08-17**  
> 阶段目标：建立可重复实验的 Neural Motion 工作站，并形成足以支撑 Flowform M2～M4 的参考基线。

## 1. M1 目标已经从“跑三个参考项目”升级

初版 M1 只计划 ARDY、MotionBricks、Kimodo。经过对 CAMDM、MotionBricks 最新实现线索以及 UE5.8 AnimGen / Control Operators 的研究，M1 现在需要更明确地区分：

```text
可完整训练复现
可运行但训练不可复现
可做最小训练烟雾测试
只做架构 / 能力证据
```

M1 的最终成果不是“我们跑过很多 Demo”，而是：

> **知道哪些架构真的能在本地完整跑通、它们各自在解决什么问题，并据此冻结 Flowform 首版 Control / Model / Runtime 边界。**

## 2. 参考优先级

### S — 必须优先完成

#### AnimGen Player

```text
Official Sample
→ Pretrained Runtime
→ Official Training Data
→ Local Re-train
→ Local Model Runtime
→ Comparison
```

这是当前最重要的编辑器内训练闭环 Baseline。

#### AnimGen NPC

只做：

```text
Pretrained Runtime
Control / Style / Prop Architecture
LOD / Scalability
Capability Record
```

不做训练复现，因为官方 Sample 未提供 NPC Training Data。

#### Control Operators Python Reference

跑通简化参考实现：

```text
Dataset
→ Pose Representation / AutoEncoder
→ Control Operators
→ Flow-Matching Controller
→ Interactive Runtime
```

它用于理解算法最小闭环，不等同于 Epic / 论文原始生产实现。

### A — 重要架构对照

- **ARDY**：Trajectory / Anchor / Prediction / Stop-Recover / Replan；
- **CAMDM**：Future Chunk / Buffer / Lazy Replan / Trajectory Fusion / Diffusion / Style；
- **MotionBricks**：Motion Representation / multi-stage architecture / async runtime / large motion corpus。

### B — 数据方向

- **Kimodo**：Motion Generation / Teacher / Synthetic Dataset，不阻塞 M1 Core Gate。

## 3. 本阶段仍然不做

- 不正式开发 Flowform 自有模型；
- 不正式开发 Flowform Lab；
- 不写 UE / Unity / Godot Bridge；
- 不写跨引擎 C++ Runtime；
- 不因为 AnimGen 很强就直接复制其架构；
- 不因为 CAMDM 很强就直接改成 Diffusion；
- 不猜测 AnimGen NPC 未公开的数据规模 / 训练方法；
- 不为了“全绿”而对不可获得项目伪造复现结论。

原则：

> **Baseline first. Evidence before architecture.**

## 4. 工作站基础

M1-S1 现有计划保持有效：Windows / WSL2 / NVIDIA / CUDA / Python / PyTorch / 5090D / EnvironmentReport。

现有执行索引：[`m1-s1/README.md`](m1-s1/README.md)

环境通过标准：

```text
Windows Dev Base       PASS
WSL2                   PASS
GPU / Driver           PASS
Python Environment     PASS
PyTorch CUDA           PASS
GPU Tensor Smoke Test  PASS
EnvironmentReport      PASS
```

对 UE5.8 AnimGen，允许使用 Windows / Unreal 官方训练路径，不强制为了统一而迁入 WSL2。参考项目遵循其官方、最少摩擦的运行环境；Flowform 自有训练环境仍优先保持可脚本化与可复现。

---

# Sprint 1｜Workstation Baseline
## 2026-08-10 ～ 2026-08-16

沿用现有 `docs/plans/m1-s1/`。

完成后输出：

```text
M1-S1: PASS
Next: M1-S2 AnimGen Official Sample Baseline
```

> 原计划中的 `Next: ARDY Bring-up` 已被最新研究结论替换。

---

# Sprint 2｜AnimGen 官方样板基线
## 建议：2026-08-17 ～ 2026-08-20

详细计划：[`m1-s2/README.md`](m1-s2/README.md)

### 目标

1. 下载 / 打开 UE5.8 AnimGen 官方 Sample；
2. 运行 Player Controller；
3. 运行 NPC Controller；
4. 使用官方数据、默认配置重新训练 Player；
5. 用本地训练产物重新运行 Player；
6. 完成 Pretrained vs Local Re-trained 对比。

### 记录

```text
UE Version
Sample Version
GPU / Driver
Training Config
Training Duration
Peak VRAM
Output Size
Runtime Cost
A01-A11 Scenario
Visual Notes
```

### Gate

```text
AnimGen Sample Open             PASS
Player Pretrained Runtime       PASS
NPC Capability Record           PASS
Player Official Re-train        PASS
Local Re-trained Runtime        PASS
Baseline Report                 PASS
```

NPC 明确记录：

```text
Training Data: NOT PROVIDED
Training Reproduction: NOT AVAILABLE
```

---

# Sprint 3｜Control Operators Python Reference
## 建议：2026-08-21 ～ 2026-08-23

详细计划：[`m1-s3/README.md`](m1-s3/README.md)

### 目标

跑通公开简化实现，重点理解：

- Control Operators 如何组合；
- Control Encoder 如何与 Controller 分离；
- Pose / Latent 表示；
- Flow Matching 训练；
- Autoregressive frame-by-frame Runtime；
- 最小模型的参数规模与实际训练成本。

### Gate

```text
Reference Environment       PASS
Dataset Loading             PASS
Training Smoke / Baseline   PASS
Interactive Runtime         PASS
Architecture Notes          PASS
```

如果上游仓库许可证未明确，不把代码复制进 Flowform；只在 `third_party/` 或外部 workspace 中运行，并保留来源记录。

---

# Sprint 4｜ARDY Control / Replan Baseline
## 建议：2026-08-24 ～ 2026-08-27

ARDY 继续承担“Future Intent / Replan”对照角色。

固定 Scenario：

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

重点记录：

- Target Velocity；
- Target Heading；
- History；
- Prediction Horizon；
- Trajectory / Anchor；
- Replan Trigger；
- Stop-Recover 行为；
- 推理与游戏线程成本统计口径。

Gate：

```text
ARDY_RUNTIME = PASS
ARDY_A01_A09 = PASS
ARDY_REPLAN_CARD = PASS
```

---

# Sprint 5｜CAMDM + MotionBricks 对照
## 建议：2026-08-28 ～ 2026-09-01

## CAMDM

M1 要求：

- 官方预训练 / Unity Runtime 跑通；
- Future Chunk / Buffer / Replan 行为可观察；
- Style 控制可验证；
- Dataset → Training 至少做 smoke test；
- 完整长训练可延后，不阻塞 M1。

状态允许：

```text
Runtime PASS
Training Smoke PASS
Full Training DEFERRED
```

## MotionBricks

优先确认：

- 可获得 Source / Implementation 到底是什么；
- Motion Representation；
- 多阶段网络关系；
- GASP 等语料的角色；
- Runtime 是否真正异步；
- “Game Thread 0.002ms”等公开性能数字的统计边界。

若缺完整训练代码 / 数据：

```text
Reproducibility = R0/R1
Architecture Evidence = PASS
Training Reproduction = NOT CLAIMED
```

不能用社交视频描述替代训练复现。

---

# Sprint 6｜Kimodo / Synthetic Data Triage
## 建议：2026-09-02 ～ 2026-09-04

只回答：

1. 输入条件是什么；
2. 能否控制 Trajectory / Constraint；
3. 能否批量生成；
4. Skeleton / Output Format；
5. 能否进入 Flowform Canonical Dataset；
6. 输出与训练来源许可证是否适合未来开源 / 商用。

如不影响 M3 数据路线，可标记：

```text
KIMODO = DEFER_TO_M3
```

不阻塞 M1。

---

# Sprint 7｜统一 Reference Decision
## 建议：2026-09-05 ～ 2026-09-09

这是 M1 真正的收口任务。

建立统一对照：

| Dimension | AnimGen | Control Operators Ref | ARDY | CAMDM | MotionBricks |
|---|---|---|---|---|---|
| Pose Representation | | | | | |
| Control Representation | | | | | |
| Generation Unit | | | | | |
| Runtime Strategy | | | | | |
| Replan | | | | | |
| Style | | | | | |
| Interaction / Props | | | | | |
| Training Reproducibility | | | | | |
| Runtime Cost | | | | | |
| Multi-character Scaling | | | | | |
| License Boundary | | | | | |

最终输出：

```text
M1_REFERENCE_DECISION.md
```

必须回答：

1. M4 V0-A 是否采用 Latent Autoregressive Flow Matching？
2. V0-B 的 deterministic baseline 用什么结构？
3. Pose AutoEncoder 是否默认启用？
4. Flowform Control Schema V0 包含哪些 Element？
5. Future-chunk Backend 在 M4 做到什么程度，还是 M5 才正式实现？
6. Runtime 如何统一 NextPose 与 FutureChunk？
7. M2 Lab 第一版需要显示哪些 Debug Channel？

## 5. 统一实验记录

每次实验至少记录：

```text
Experiment ID
Date
Flowform Commit
Reference / Commit / Sample Version
Environment
Model / Dataset
Config
Training Duration
Peak VRAM
Runtime Metrics
Scenario
Result
Notes
Screenshots / Video
License Status
Reproducibility Level
```

### Reproducibility Level

```text
R3 FULL      Training + Data + Runtime 可重复
R2 BASELINE  核心闭环可重复，但生产数据 / 部分实现缺失
R1 RUNTIME   只能运行预训练结果
R0 EVIDENCE  只能做论文 / 代码片段 / 视频 / 架构分析
```

## 6. TestReport

统一目录：

```text
reports/
└─ YYYY-MM-DD_REFERENCE_SCENARIO/
   ├─ environment.json
   ├─ source.json
   ├─ config.json
   ├─ runtime.log
   ├─ metrics.json
   ├─ notes.md
   ├─ screenshots/
   └─ video/
```

大文件不进 Git。仓库提交 Report Summary、Reference Card 与必要配置。

## 7. M1 最终 Gate

M1 不再要求“所有参考都完整训练”。正确 Gate 是：

```text
Environment / GPU                  PASS
Experiment Report Pipeline         PASS
AnimGen Player Re-train            PASS
AnimGen NPC Boundary Recorded      PASS
Control Operators Reference        PASS
ARDY Control / Replan Baseline     PASS
CAMDM / MotionBricks Evidence      PASS or DOCUMENTED_DEFER
Reference Matrix                   PASS
M1 Architecture Decision           PASS
```

通过后进入 M2。

## 8. M1 完成后禁止遗留的模糊说法

不得出现：

- “AnimGen 整套系统我们已经复现”——除非明确限定 Player Baseline；
- “AnimGen NPC 可以重训”——当前官方 Sample 不支持；
- “MotionBricks 0.002ms 比 AnimGen 1ms 快几百倍”——统计口径未统一；
- “CAMDM 证明 Flowform 应该改用 Diffusion”——它只证明一种 Backend 可行；
- “Control Operators Reference 就是 Epic AnimGen 源码”——公开 Reference 是简化实现；
- “M4 一定是 Transformer”——该约束已取消。
