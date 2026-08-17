# AnimGen 官方样板基线计划

> Reference: Unreal Engine 5.8 Experimental AnimGen  
> Flowform Priority: **S**  
> Baseline Status: Pending local execution  
> Updated: 2026-08-17

## 1. 目标

本任务不是“复刻 Epic 最强 NPC Controller”，而是建立可信的 AnimGen 官方基线：

```text
下载官方 Sample
      ↓
运行预训练 Player
      ↓
运行预训练 NPC
      ↓
使用官方数据重新训练 Player
      ↓
使用本地训练产物重新运行 Player
      ↓
记录与官方结果差异
```

第一轮禁止擅自改网络、删数据、换 Control、重构工程。

## 2. 必须区分两个示例

### A. Player Controller — 可训练 Baseline

官方示例描述明确表示：

- basic walking locomotion；
- aiming；
- firing a pistol；
- all training data is provided；
- can be re-trained within the sample。

因此它是 Flowform M1 的**正式训练复现基线**。

### B. NPC Controller — 只做能力参考

官方示例描述明确表示：

- walking locomotion；
- very large number of styles；
- props；
- pre-trained；
- training data is not provided；
- cannot be re-trained in editor。

因此：

```text
AnimGen NPC
= Runtime / Architecture / Scalability Reference
!= Reproducible Training Baseline
```

任何 Flowform 文档都不得写“我们可以完整复现 AnimGen NPC 训练”。

## 3. 第一轮环境记录

至少记录：

```text
Date
Flowform Commit
Unreal Engine Version
AnimGen Plugin Version / Engine Build
Sample Version
Windows Version
GPU
Driver
CPU
RAM
Project Path
```

训练时追加：

```text
Training Dataset / AnimDatabase
AutoEncoder Config
Controller Config
Training Device
Training Start / End
Training Duration
Peak VRAM
Peak System RAM
Output Asset / Model Size
Warnings / Errors
```

Runtime 追加：

```text
Controller
LOD / Quality Setting
Evaluation Period
FPS
Game Thread
Anim Thread / Worker if observable
Inference / Controller cost if observable
Visual Notes
```

不要为了得到某个数字而修改引擎 Profiling 路径；第一轮优先记录官方样板能够直接观测的数据。

## 4. Phase A — 下载与打开官方 Sample

- [ ] 从 Epic 官方渠道获取 AnimGen Sample；
- [ ] 使用与样板匹配的 UE5.8 版本创建 / 打开工程；
- [ ] 确认 AnimGen Experimental Plugin 正常启用；
- [ ] 不修改 Sample Content；
- [ ] 保存项目版本与来源信息。

Gate：

```text
ANIMGEN_SAMPLE_OPEN = PASS
```

## 5. Phase B — 运行预训练 Player

固定记录：

- Locomotion；
- Aim；
- Fire pistol；
- Start / Stop；
- 90° / 180°；
- Rapid Input Change；
- Root / Foot 视觉问题；
- Runtime 性能。

建议复用 Flowform Motion Test Protocol：

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
A10 Aim While Moving
A11 Fire While Moving
```

Gate：

```text
ANIMGEN_PLAYER_PRETRAINED = PASS
```

## 6. Phase C — 运行预训练 NPC

NPC 只做观察与结构研究：

- Style 数量与切换方式；
- Prop 类型；
- Control / Behavior 组织；
- LOD；
- Evaluation Period；
- Runtime 成本；
- Model / Asset 组织；
- Style / Prop 切换的连续性；
- 多角色 Scaling（若 Sample 直接支持）。

必须在报告中写：

```text
Training Data: NOT PROVIDED
Training Reproduction: NOT AVAILABLE
Reproducibility: R1 RUNTIME
```

Gate：

```text
ANIMGEN_NPC_CAPABILITY_RECORD = PASS
```

## 7. Phase D — 官方默认配置重新训练 Player

原则：

> **只做官方默认 Baseline。**

- [ ] 确认 Player 的训练数据来源；
- [ ] 记录 AutoEncoder 设置；
- [ ] 记录 Controller 设置；
- [ ] 启动 AutoEncoder / Controller 所需训练；
- [ ] 不手工“优化”参数；
- [ ] 保存完整训练日志；
- [ ] 保存 TensorBoard / Training UI 数据（若有）；
- [ ] 保存训练产物；
- [ ] 记录耗时、VRAM、错误 / Warning。

训练失败时，不通过修改大量参数“强行跑通”；应先记录：

```text
Expected Official Path
Actual Failure
Environment
Log
Potential Cause
```

Gate：

```text
ANIMGEN_PLAYER_RETRAIN = PASS
```

## 8. Phase E — 使用本地训练产物运行

重新执行 Phase B 的 A01～A11。

对比：

| Item | Epic Pre-trained | Local Re-trained |
|---|---|---|
| Idle Stability | | |
| Start | | |
| Stop | | |
| Stop-Recover | | |
| 90 Turn | | |
| 180 Turn | | |
| Rapid Change | | |
| Aim | | |
| Fire | | |
| Foot Slide | | |
| Runtime Cost | | |

重点问题不是“必须完全一样”，而是：

> 在相同 Sample、相同官方数据、默认训练流程下，本地是否能得到同等级别、可实际运行的 Controller？

## 9. 输出目录

建议：

```text
reports/
└─ animgen-baseline-YYYYMMDD/
   ├─ environment.json
   ├─ sample_version.txt
   ├─ pretrained_player/
   ├─ pretrained_npc/
   ├─ training/
   │  ├─ config/
   │  ├─ logs/
   │  └─ artifacts/
   ├─ retrained_player/
   ├─ screenshots/
   ├─ video/
   └─ notes.md
```

大文件不提交 Git；仓库只提交结构化结论与必要的小型配置 / 报告摘要。

## 10. Baseline 完成后的第二轮实验

只有第一轮通过后才进入：

### Ablation A — 数据量

```text
100%
50%
25%
10%
```

确认训练数据量与动作质量 / 泛化的关系。

### Ablation B — Control

增加 / 删除一个 Control，观察：

- 是否需要全部重训；
- Control Encoder / Schema 如何变化；
- Runtime API 如何变化。

### Ablation C — Pose / AutoEncoder

改变 Latent / Feature 设置，观察：

- Reconstruction；
- Motion Continuity；
- Controller 学习难度；
- Runtime 成本。

### Ablation D — LOD / Distillation

比较不同 Runtime Quality Tier 的：

- 质量；
- 单角色成本；
- 多角色 Scaling。

## 11. Flowform 最终要从本实验回答的问题

1. Flow-Matching Autoregressive Controller 是否适合作为 Flowform V0-A？
2. Pose AutoEncoder 是否应成为 Base Model 的默认组件？
3. Control Schema 应如何映射到 Flowform 自己的类型系统？
4. Runtime LOD / Distillation 是否值得在 v1.0 正式支持？
5. 相比 ARDY / CAMDM 的 Future-chunk 方案，逐帧生成的控制响应和成本如何？
6. 用户需要多少数据，才能训练一个“够用”的基础 locomotion Controller？
7. Epic 样板中哪些关键能力依赖未公开的 NPC 数据 / 训练工程，不能据此推断 Flowform 很容易达到同等规模？

## 12. Sources

- Epic AnimGen Tutorial: https://dev.epicgames.com/community/learning/tutorials/eGME/unreal-engine-animgen
- Epic AnimGen API: https://dev.epicgames.com/documentation/unreal-engine/API/Plugins/AnimGen
- UAnimGenController: https://dev.epicgames.com/documentation/unreal-engine/API/Plugins/AnimGen/UAnimGenController
- Control Operators overview: https://theorangeduck.com/page/control-operators-interactive-character-animation
