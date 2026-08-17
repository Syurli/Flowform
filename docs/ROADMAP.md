# Flowform 一年开发路线图

> 项目：**流形 Flowform**  
> 技术定位：**Neural Motion Framework**  
> 品牌归属：**A BAIGE Project**  
> 周期：**2026-08-10 ～ 2027-08-09**  
> 本次校准：**2026-08-17**

## 1. 年度产品目标

第一年不以论文复现或单一神经网络 Demo 为终点，而是交付一个真正可训练、可调试、可跨引擎运行的 Neural Motion Framework。

目标用户为独立开发者与中小型游戏团队。

理想用户流程：

```text
安装 Flowform
    ↓
导入自己的动作
BVH / FBX / Mocap
    ↓
Skeleton Mapping / Retarget
    ↓
自动预处理 + Dataset Audit
    ↓
配置 Control Schema
    ↓
训练 / 微调 Controller
    ↓
Flowform Lab 实时试玩与 Benchmark
    ↓
Export Model Package
    ↓
Unreal / Unity / Godot / Web
    ↓
实时 Neural Motion
```

## 2. v1.0 范围

第一代重点覆盖：

- Locomotion；
- Start / Stop；
- 加减速；
- 任意方向转向；
- Strafe / Backward；
- Rapid Input Change；
- Continuous Transition；
- Style-conditioned Motion；
- 基础 Aim / Prop / Overlay Control 接口；
- 为 Traversal / Interaction 留正式 Control 接口。

第一年不追求神经网络替代所有动画系统。攻击、换弹、技能、处决和严格 Timing 的动画仍允许传统 Animation / Montage / Layered Animation 接管。

## 3. 经最新研究校准后的核心架构

Flowform 不再假设“所有模型都一次生成 Future Motion Chunk”，也不把某一个 Transformer 结构写死为产品核心。

```text
Motion Data
   ↓
Canonical Skeleton / Motion Representation
   ↓
Pose Encoder / Decoder（可选）

Gameplay / Designer Intent
   ↓
Composable Control Schema
   ↓
Control Encoder

           ┌────────────────────────────┐
           │      IMotionModel          │
           └─────────────┬──────────────┘
                         │
          ┌──────────────┴──────────────┐
          │                             │
Autoregressive Backend          Future-chunk Backend
frame-by-frame                  buffered generation
Flow Matching / etc.            Transformer / Diffusion / etc.
          │                             │
      Next Pose                 Future Motion Chunk
          └──────────────┬──────────────┘
                         ↓
                  Flowform Runtime
                         ↓
                 Engine Bridge / Web
```

核心原则：

1. **Motion Representation、Control Schema、Model、Runtime Strategy 相互解耦。**
2. Control 不以固定超长向量为唯一表示，而以可组合 Schema 描述 Trajectory、Facing、Style、Target、Optional State、Interaction 等语义。
3. Runtime 必须能够同时容纳 frame-by-frame autoregressive 与 future-buffer / replan 两类模型。
4. Model Package 需要声明模型能力、历史长度、预测方式、Runtime Strategy、LOD / Quality Tier 与许可证信息。
5. Unreal 是首个正式引擎落地，但不能成为核心架构前提。

详见 [`ARCHITECTURE.md`](ARCHITECTURE.md)。

## 4. 参考系统分级

| Priority | Reference | 主要回答的问题 |
|---|---|---|
| S | AnimGen / Control Operators | 如何让设计师组合 Control？如何在 Latent Pose 上训练自回归 Flow-Matching Controller？如何在编辑器训练、蒸馏和分 LOD？ |
| A | ARDY | 玩家意图、未来轨迹、Anchor、响应和 Replan 应如何组织？ |
| A | CAMDM | Future Motion Buffer、Lazy Replan、Trajectory Fusion、Style 与实时 Diffusion 如何组织？ |
| A | MotionBricks | Motion Representation、模块化训练、异步 Runtime 与大规模动作语料如何组织？ |
| B | Kimodo | Synthetic / Teacher Motion 是否能降低数据获取成本？ |

参考优先级不等于最终技术选型。Flowform 的技术选型必须由可重复实验结果决定。

## 5. 12 个月里程碑

| 月份 | 阶段 | 核心交付 | 状态目标 |
|---|---|---|---|
| M1 | 工作站与参考系统 | AnimGen / Control Operators + ARDY / CAMDM / MotionBricks 基线 | 形成可信技术基线 |
| M2 | Flowform Lab Alpha | 多参考系统观察 / 回放 / 控制与统一实验 UI | 第一套产品原型 |
| M3 | 数据与 Control 系统 | Canonical Skeleton + Dataset Pipeline + Control Schema | 摆脱特定仓库格式 |
| M4 | **Flowform Controller V0** | 自有 Latent / Control / Controller 最小闭环 | **网页运行自己的 Controller** |
| M5 | Runtime Strategy V1 | Autoregressive + Future-chunk 统一 Runtime | 操控达到可玩 |
| M6 | Benchmark 与数据质量 | Research + Gameplay Benchmark、Dataset Audit | 可系统迭代 |
| M7 | Style / Custom Training | Style Embedding / Adapter / Control | 用户可训练风格 |
| M8 | Flowform Runtime Alpha | ONNX + C++ Runtime + Model Package | 脱离 Python Runtime |
| M9 | Flowform UE Alpha | Unreal 实装 | 第一个正式游戏引擎 |
| M10 | 跨引擎 Beta | Unity + Godot Bridge | 架构验证完成 |
| M11 | Open Source Beta | Training UX / 示例 / CI / 文档 | 外部开发者可使用 |
| M12 | **v1.0** | 开源发布 + Base Model + Samples | 第一代产品完成 |

---

# M1｜工作站与参考系统基线
**2026-08-10 ～ 2026-09-09**

目标：建立可重复实验工作站，并通过可复现 Baseline 回答 Flowform M2～M4 的关键架构问题。

### M1 核心参考

1. **AnimGen Player Controller**：运行官方预训练版本；使用示例自带训练数据按默认配置重新训练；使用本地训练产物重新运行；记录训练时间、VRAM、产物、Runtime 与动作表现。
2. **AnimGen NPC Controller**：只运行、观察、记录。由于官方未提供训练数据，不把它列为可复现训练 Baseline。
3. **Control Operators Python Reference**：跑通公开的简化参考实现，理解 Control Operator、Pose Latent 与 Flow-Matching Autoregressive Controller 的最小闭环。
4. **ARDY**：验证 Player Intent / Trajectory / Anchor / Replan。
5. **CAMDM**：验证 Future Chunk / Lazy Replan / Trajectory Fusion / Style / Diffusion Runtime。
6. **MotionBricks**：验证可获得实现中的 Motion Representation / Training / Runtime 架构；若公开实现或训练资料不足，允许以 Architecture Evidence Card 代替完整训练。
7. **Kimodo**：Teacher / Synthetic Motion 评估，不作为 M1 核心 Gate 阻塞项。

### M1 Gate

```text
Environment / GPU                  PASS
Experiment Report Pipeline         PASS
AnimGen Player Runtime             PASS
AnimGen Player Re-train            PASS
AnimGen NPC Capability Record      PASS
Control Operators Reference        PASS
ARDY Control / Replan Baseline     PASS
A-level Architecture Comparison    PASS
Reference Decision Report          PASS
```

CAMDM / MotionBricks 的完整长训练不是 M1 的强制条件；要求至少完成预训练推理、最小训练烟雾测试或形成明确的不可复现 / 延后原因记录。

详细计划：[`plans/M1_WORKSTATION_BASELINE.md`](plans/M1_WORKSTATION_BASELINE.md)

---

# M2｜Flowform Lab Alpha
**2026-09-10 ～ 2026-10-09**

目标：建立第一套属于 Flowform 的 Web 产品原型，但不再让 Lab 绑定 ARDY 单一 Provider。

第一版能力：

- Pose Viewer；
- Root / Trajectory Viewer；
- Control Inspector；
- Predicted / Desired / Blended Trajectory；
- Frame-by-frame 与 Future-chunk 两种生成结果观察；
- Foot Contact；
- Runtime Timing；
- Model / Reference Compare；
- Experiment Replay；
- Scenario Runner。

建议接口从简单的 `Predict(history, intent) -> future` 升级为 Capability-based：

```text
IMotionModel
├─ DescribeCapabilities()
├─ Reset(PoseState)
├─ SetControls(ControlSet)
├─ Step(dt)
└─ GetGenerationResult()
```

`GenerationResult` 可以是 Next Pose，也可以携带 Future Motion Chunk。

**Gate B：同一个 Flowform Lab 可以观察至少两类不同 Runtime Strategy，而 UI 不依赖具体模型。**

---

# M3｜数据体系 + Control Schema
**2026-10-10 ～ 2026-11-09**

建立 Flowform 自己的数据与控制语义层。

### Canonical Motion

第一版 Skeleton 聚焦主要身体骨骼，Face / Finger 不作为 v1.0 必需项。

核心数据：

```text
SkeletonDefinition
MotionFrame
MotionClip
MotionDataset
PoseFeature
RootState
FootContact
```

特征候选至少覆盖 Rotation6D、Joint Position / Velocity、Root Position / Velocity / Heading、Contact，并通过实验决定 Pose AutoEncoder 输入。

### Control Schema

第一批语义：

```text
Trajectory
Velocity
Facing
Style / Tag
Optional
Target
Interaction
Prop / Equipment
Multi / Composite
```

并明确区分：

```text
Ground Truth Motion != Runtime Control Intent
```

数据预处理必须支持 Control Augmentation，例如轨迹平滑、噪声、突然改向、Stop-Recover 与非完美玩家输入。

**Gate C：训练数据与控制数据不再依赖任何单一参考项目内部格式。**

---

# M4｜Flowform Controller V0
**2026-11-10 ～ 2026-12-09**

M4 不再预设“必须是 5M～20M Small Transformer”。目标改为：

> **用最小、自有、可解释的 Controller 完成 Dataset → Training → Runtime → Web → Benchmark 闭环。**

优先实验：

### V0-A｜Latent Autoregressive Flow Controller

```text
Pose
 ↓
Pose AutoEncoder
 ↓
Previous Pose Latent
 +
Encoded Control
 +
Flow State / Noise
 ↓
Flow-Matching Controller
 ↓
Next Pose Latent
 ↓
Decoder
 ↓
Next Pose
```

参考 Control Operators / AnimGen 的思想，但独立实现。

### V0-B｜Deterministic Baseline

使用同一 Pose Representation 与 Control Schema，训练一个更简单的 deterministic predictor，作为质量、稳定性、成本与调试难度对照。

只有实验显示 sequence Transformer / future-chunk 更适合时，才把它提升为首版主模型。

重点 Scenario：Idle→Walk、Idle→Run、Run→Stop、Stop-Recover、90°、180°、Strafe、Acceleration / Deceleration、Rapid Direction Change。

**Gate D：Flowform Lab 能完全脱离参考系统运行 Flowform 自有 Controller。**

---

# M5｜Runtime Strategy V1
**2026-12-10 ～ 2027-01-09**

统一两类运行时：

### Autoregressive

```text
Current Pose + Controls
        ↓
Generate Next Pose
        ↓
Update State
```

### Future-chunk

```text
History + Controls
        ↓
Generate Future Chunk
        ↓
Motion Buffer
        ↓
Consume / Replan
```

公共 Runtime 负责：

- Root / Pose；
- Contact；
- Correction；
- Async Inference；
- Generation ID；
- Reset / Teleport；
- LOD / Evaluation Period；
- Fallback；
- Runtime Metrics。

**Gate：动作具有可接受的游戏操控感，并且 Runtime 不绑定一种模型策略。**

---

# M6｜Benchmark 与数据质量体系
**2027-01-10 ～ 2027-02-09**

建立两层 Benchmark。

### Research Metrics

- Foot Sliding；
- Acceleration / Jerk；
- Trajectory Error；
- Orientation Error；
- Style Accuracy；
- Diversity；
- Pose / Root Error；
- 适用时加入 FID 等分布指标。

### Gameplay Metrics

- Input Response；
- Stop Distance；
- Replan / Recovery Latency；
- 90° / 180° Turn；
- Rapid Input Change；
- Root Drift；
- Pose Jitter；
- Frame Time；
- CPU / GPU Cost；
- 多角色 Scaling。

同时增加 Dataset Audit：覆盖率、方向 / 速度 / Transition / Style / Prop 分布与缺失提示。

---

# M7｜Style 与用户自定义训练
**2027-02-10 ～ 2027-03-09**

目标：**Bring Your Own Motion Style**。

研究：

- Style / Tag Control；
- Style Embedding；
- Lightweight Adapter；
- 少量动作微调；
- Style Transition；
- Dataset Balance。

Flowform Training 应能够告诉用户“缺什么数据”，而不是只给一个 Train 按钮。

**Gate E：同一控制输入下，Base + 至少两个用户风格可以稳定区分。**

---

# M8｜Engine-independent Runtime Alpha
**2027-03-10 ～ 2027-04-09**

建立 C++ Runtime 与统一 Model Package。

候选内容：

```text
Model.flowform/
├─ model.json
├─ skeleton.json
├─ controls.json
├─ normalization.json
├─ pose_encoder.onnx      # optional
├─ pose_decoder.onnx      # optional
├─ control_encoder.onnx   # optional
├─ controller.onnx
├─ lod/                   # optional
└─ license.json
```

Backend 至少支持 ONNX Runtime CPU / CUDA；TensorRT 或其他高性能 Backend 由 Benchmark 决定，不在文档阶段写死。

**Gate F：Python 与 C++ Runtime 输出在允许容差内一致。**

---

# M9｜Flowform UE Alpha
**2027-04-10 ～ 2027-05-09**

实现 Unreal Bridge：Model Load、Skeleton Mapping、Movement / Control 输入、Async Inference、Pose Output、Debug Draw、Fallback、LOD。

AnimGen 是重要 UE 产品化参考，但 Flowform UE 必须通过 Flowform Runtime / Model Package 工作，不复制 AnimGen Engine Code，也不让 Flowform 核心依赖 UE。

**Gate G：标准 UE Demo 完成 Flowform 自有 Controller Locomotion。**

---

# M10｜跨引擎验证
**2027-05-10 ～ 2027-06-09**

同一 Model Package 至少在：

```text
Flowform Lab
Flowform UE
Flowform Unity
Flowform Godot
```

完成基本一致的 locomotion。

**Gate H：Cross-engine Architecture Validated。**

---

# M11｜Open Source Beta
**2027-06-10 ～ 2027-07-09**

停止扩张大功能，重点验证陌生开发者是否能够：安装、导入数据、训练、Benchmark、导出、接入 UE，并理解许可证边界。

完善 `flowform doctor`、Sample Dataset、Base Model、Training Preset、CI、模型卡、Dataset Card、License Matrix。

---

# M12｜v1.0 Release
**2027-07-10 ～ 2027-08-09**

只做 Bug Fix、Performance、Packaging、Regression、Documentation 与 Release。

最终目标仍然是：

> **一个开源、可训练、可调试、跨引擎、且不被单一神经网络结构定义的 Neural Motion Framework。**

## 6. 年度关键 Gate

1. **Reference Gate**：AnimGen Player 可本地重训，Control Operators Reference 可运行，主要参考系统形成统一对照结论。
2. **Lab Gate**：Flowform Lab 不绑定某一种模型或 Runtime Strategy。
3. **Own Controller Gate**：自有 Controller 在 Web 中实时运行。
4. **Gameplay Gate**：Start / Stop / Rapid Change / Turn 达到游戏可用。
5. **Custom Training Gate**：用户动作 / Style 可训练。
6. **Runtime Gate**：C++ Runtime 与 Model Package 独立于引擎。
7. **Cross-engine Gate**：同一模型包运行于多个引擎。

## 7. 路线调整规则

后续发现新论文 / 新 Demo 时按以下顺序处理：

```text
发现参考
  ↓
确认来源 / 许可 / 可复现性
  ↓
建立 Reference Card
  ↓
跑 Baseline 或记录不可复现边界
  ↓
与现有 Scenario / Benchmark 对照
  ↓
形成 Architecture Decision
  ↓
必要时更新 Roadmap
```

不再因为单个效果视频直接重写核心架构。
