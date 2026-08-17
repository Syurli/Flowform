# 流形 Flowform

**Neural Motion Framework**  
*A BAIGE Project*

Flowform（流形）是一个面向游戏角色运动的开源、可训练、跨引擎 Neural Motion Framework。

项目目标不是把某一种神经网络包装成插件，也不是在 v1.0 中替代所有传统动画系统；第一阶段聚焦实时 Locomotion、Start / Stop、加减速、任意方向转向、Strafe / Backward、连续过渡与 Style-conditioned Motion，并为 Traversal、Interaction、Combat Overlay 等后续能力保留正式接口。

## Core Idea

Flowform 当前把产品核心定义为五个相互解耦的层次：

```text
Motion Representation
        +
Composable Control Schema
        +
Pluggable Neural Motion Models
        +
Engine-independent Runtime
        +
Training / Motion Lab
```

网络结构不是产品本身。Flowform 需要同时容纳不同模型与运行时策略，例如：

- **Autoregressive / frame-by-frame**：参考 AnimGen / Control Operators 的 Flow-Matching Controller；
- **Future-chunk / buffered generation**：参考 CAMDM、ARDY 等未来动作预测与 Replan 路线；
- 后续允许继续增加 Deterministic Transformer、Diffusion、MotionBricks-style Backend 等实现。

## Product Architecture

- **Flowform Lab** — Web 运动验证、Runtime Debug、Dataset Inspector、Control Inspector、Model Compare、Training Monitor、Style Preview 与 Benchmark。
- **Flowform Training** — 动作导入、Skeleton Mapping / Retarget、预处理、Control Schema、训练 / 微调、Benchmark、蒸馏 / LOD 与模型导出。
- **Flowform Runtime** — Engine-independent Runtime，统一处理模型加载、输入控制、异步推理、Pose / Root / Contact、Buffer / Replan 或 Autoregressive Update 等运行时行为。
- **Flowform Model Package** — 保存 Skeleton、Normalization、Pose Encoder / Decoder、Control Encoder、Controller、Runtime Strategy、Quality Tier / LOD 与许可证元数据。
- **Engine Bridges** — Flowform UE、Flowform Unity、Flowform Godot，以及 Web Runtime。

训练系统与引擎通过 **Canonical Skeleton + Control Schema + Model Package** 解耦，目标是让同一模型包能够跨引擎使用。

## Current Research Priority

截至 2026-08-17，参考系统按对 Flowform 的直接价值分级：

| Priority | Reference | 主要用途 |
|---|---|---|
| S | AnimGen / Control Operators | Control Schema、Pose Latent、Flow Matching、自回归 Controller、训练 UX、LOD / Distillation |
| A | ARDY | 玩家意图、Trajectory / Anchor、响应、Replan |
| A | CAMDM | Future Motion Buffer、Lazy Replan、Trajectory Fusion、Diffusion、Style |
| A | MotionBricks | Motion Representation、模块化训练与高性能 Runtime 参考 |
| B | Kimodo | Motion Generation、Teacher / Synthetic Dataset 方向 |

**AnimGen 的两个官方示例必须区别对待：** Player Controller 提供训练数据，可作为训练闭环基线；高级 NPC Controller 不提供训练数据，只作为 Runtime、Control 规模与能力上限参考，不能宣称其训练可复现。

详见 [`docs/reference/REFERENCE_MATRIX.md`](docs/reference/REFERENCE_MATRIX.md)。

## v1.0 Target

计划周期：**2026-08-10 ～ 2027-08-09**。

第一年目标：交付一个可实际开源使用的跨引擎 Neural Motion Framework，包括：

- 自有实时 Locomotion Controller，不依赖参考系统推理；
- 可组合 Control Schema；
- 至少一种自有 Autoregressive Controller，并保留 Future-chunk Backend；
- Start / Stop / Replan / Rapid Input Change；
- 连续方向与速度控制；
- Style Training 与用户自定义动作训练；
- Flowform Lab Web；
- Flowform Training；
- Engine-independent C++ Runtime；
- ONNX Runtime / CUDA，并为高性能 Backend 与模型 LOD 留出路径；
- Windows / Linux；
- Flowform UE 正式支持；
- Flowform Unity / Godot Beta；
- Model Package、自动化环境、测试、CI 与文档。

完整路线图见 [`docs/ROADMAP.md`](docs/ROADMAP.md)。  
技术架构见 [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)。  
当前 M1 执行计划见 [`docs/plans/M1_WORKSTATION_BASELINE.md`](docs/plans/M1_WORKSTATION_BASELINE.md)。

## Current Phase

**M1 — Workstation & Reference Systems Baseline**  
2026-08-10 ～ 2026-09-09

M1 当前优先级已经根据最新研究调整：

1. 完成工作站环境基线；
2. **AnimGen 官方样板基线复现**：运行 Player / NPC，使用官方数据重新训练 Player，并用本地训练产物重新运行；
3. **Control Operators Python Reference**：跑通可公开训练的简化参考实现；
4. ARDY / CAMDM / MotionBricks 做架构对照与最小 Bring-up；
5. Kimodo 作为数据生成方向评估，不阻塞核心 Gate；
6. 所有实验统一记录环境、版本、训练时间、显存、模型产物、Runtime 指标和人工动作评价。

M1 的目标不是复制 Epic 的高级 NPC Controller，而是建立一个可信、可重复的 Neural Motion 参考基线，并据此冻结 Flowform M2～M4 的首版架构。

## Development Principle

Flowform 采用结果驱动研发：

- 产品负责人负责目标、操控体验、动作表现判断与验收；
- 架构、模型、训练、Runtime、实验设计与问题定位以工程验证推进；
- 不因为单篇论文或单个 Demo 直接更换产品路线；
- 参考项目先做 Baseline，再做对照实验，最后才进入 Flowform 自有实现；
- Loss 等训练指标用于诊断，最终标准仍是游戏中的动作质量、操控响应、稳定性与运行成本。

## Repository Layout

```text
Flowform/
├─ apps/
│  └─ motionlab/
├─ training/
├─ runtime/
├─ models/
├─ tools/
├─ bridges/
├─ tests/
├─ examples/
├─ scripts/
├─ configs/
├─ docs/
│  ├─ plans/
│  ├─ reference/
│  └─ licenses/
├─ third_party/
├─ workspace/      # local only
└─ reports/        # local only
```

## License Boundary

项目计划开源，但 Flowform 自有代码、参考项目代码、预训练模型、训练数据与生成数据的许可证必须分别管理。对 Unreal Engine AnimGen 等受引擎许可约束的实现，只进行研究、测试和架构对照；Flowform 跨引擎核心必须保持独立实现。第三方状态见 [`docs/licenses/third_party_registry.csv`](docs/licenses/third_party_registry.csv)。
