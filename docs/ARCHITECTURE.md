# Flowform 技术架构原则

> 状态：Architecture Direction / 2026-08-17  
> 目的：定义 Flowform 的长期边界，避免产品被某一个参考项目或某一种网络结构绑死。

## 1. Flowform 不是一个模型

Flowform 的产品核心不是 Transformer、Diffusion 或 Flow Matching 中的任何一种。

正式定义：

```text
Flowform
=
Motion Representation
+
Composable Control Schema
+
Pluggable Motion Models
+
Engine-independent Runtime
+
Training / Motion Lab
```

参考项目可以改变某一层实现，但不应轻易改变层与层之间的边界。

## 2. Motion Representation

Flowform 需要一个与训练仓库、引擎和具体角色解耦的 Canonical Motion Layer。

建议包含：

```text
SkeletonDefinition
PoseState
RootState
Joint Rotation / Position / Velocity
Contact
Attributes
MotionClip
MotionDataset
```

第一版优先处理主要身体骨骼；Face / Finger 不作为 v1.0 强制需求。

### Pose Latent

AnimGen / Control Operators 提示 Pose AutoEncoder 是一个高价值方向：先把高维 Pose + Velocity 压缩到 Latent，再让 Controller 在 Latent 中生成运动。

Flowform 因此把 Pose Encoder / Decoder 设计为**可选正式组件**，而不是写死：

```text
Raw Pose Backend
or
Latent Pose Backend
```

M4 通过对照实验决定首版 Base Model 是否强制依赖 AutoEncoder。

## 3. Composable Control Schema

这是 2026-08-17 后升级为一级架构概念的部分。

传统做法常把输入固定成：

```text
[Speed, Direction, Facing, Style, Trajectory, ...]
```

Flowform 改为描述“这个 Controller 接受哪些控制语义”。

第一批 Control Element 候选：

- `Trajectory`
- `Velocity`
- `Facing`
- `Target`
- `StyleTag`
- `Optional`
- `Either`
- `Interaction`
- `Prop`
- `Equipment`
- `Composite / Multi`
- `Uncontrolled`

示例：

```text
MilitaryLocomotion
├─ Trajectory
├─ Facing
├─ StyleTag
├─ Optional(AimTarget)
└─ Equipment(WeaponType)
```

或：

```text
InteractionController
├─ Trajectory
├─ Target(Object)
├─ Interaction(Hand / Body)
└─ Optional(Facing)
```

目标是让训练系统、Lab 和 Runtime 都认识同一份 Control Schema，而不是每个 Backend 自己定义一套输入参数。

## 4. Control Data != Ground Truth Motion

Flowform 数据系统必须明确区分：

```text
Ground Truth Motion
!=
Runtime Control Intent
```

Mocap 的真实未来轨迹通常比游戏里的摇杆、路径规划、AI Steering 更平滑、更准确。如果训练时永远输入 Ground Truth Future，Runtime 很容易遇到 OOD。

因此 Control Pipeline 应支持：

- trajectory smoothing；
- trajectory perturbation；
- noise；
- stop / recover；
- sudden direction change；
- incomplete target；
- delayed / coarse control；
- style / prop imbalance audit。

## 5. IMotionModel 不应只返回 Future Chunk

旧版抽象：

```text
Predict(History, Intent) -> FutureMotion
```

对 CAMDM / ARDY 类系统合适，但无法自然表达 AnimGen / Control Operators 的 frame-by-frame autoregressive Controller。

新方向：

```text
IMotionModel

DescribeCapabilities()
Reset(PoseState)
SetControls(ControlSet)
Step(DeltaTime)
GetGenerationResult()
```

`GenerationResult` 允许包含：

```text
NextPose
FutureMotionChunk     # optional
PredictedTrajectory  # optional
Contact               # optional
AuxiliaryState        # optional
DebugChannels         # optional
```

## 6. Runtime Strategy

### A. Autoregressive / Frame-by-frame

参考 AnimGen / Control Operators：

```text
Previous Pose / Latent
+
Current Control
+
Generation State
        ↓
Model
        ↓
Next Pose
        ↓
继续下一帧
```

优势：

- 控制变化可以在下一次生成直接进入模型；
- 不必维护长 Future Motion Buffer；
- Control Schema 与交互式输入很自然。

风险：

- 自回归误差累积；
- 每次生成成本与角色数强相关；
- Reset / Teleport / Correction / LOD 必须设计清楚。

### B. Future-chunk / Buffered Generation

参考 CAMDM、ARDY 等：

```text
History + Future Intent
        ↓
Generate N frames
        ↓
Future Motion Buffer
        ↓
Consume
        ↓
Trigger Replan when needed
```

优势：

- 一次推理可覆盖多帧；
- 能明确观察 future plan；
- 对轨迹预测、长期动作规划和异步执行友好。

风险：

- 输入变化后要处理旧 Buffer；
- Replan、Trajectory Fusion、Prediction Extension 成为核心工程问题；
- 过长 Horizon 可能降低响应感。

Flowform Runtime 必须让两种 Strategy 都成为一等公民。

## 7. Runtime Responsibilities

不论 Backend 类型，公共 Runtime 负责：

```text
Model Lifecycle
Control State
Pose State
Root State
Async Inference
Generation ID
Reset / Teleport
Correction
Contact
Fallback
LOD / Quality Tier
Evaluation Period
Metrics / Trace
Deterministic Replay (when possible)
```

Future-chunk Backend 额外需要：

```text
History Buffer
Future Motion Buffer
Replan Policy
Trajectory Fusion
```

Autoregressive Backend 额外需要：

```text
Generation State
Noise / Seed policy (if stochastic)
Autoregressive state recovery
```

## 8. Model Package

建议内容：

```text
Model.flowform/
├─ model.json
├─ capabilities.json
├─ skeleton.json
├─ controls.json
├─ normalization.json
├─ pose_encoder.onnx       # optional
├─ pose_decoder.onnx       # optional
├─ control_encoder.onnx    # optional
├─ controller.onnx
├─ lod/
│  ├─ quality_0.onnx       # optional
│  ├─ quality_1.onnx       # optional
│  └─ quality_2.onnx       # optional
└─ license.json
```

`model.json` 至少声明：

- model type；
- runtime strategy；
- history requirement；
- output mode；
- control schema version；
- skeleton version；
- deterministic / stochastic；
- preferred evaluation period；
- available quality tiers；
- runtime backend compatibility；
- package version。

## 9. LOD / Distillation

AnimGen 把 Reference Model、Distillation 和 Runtime LOD 暴露为产品能力，这一点值得 Flowform 吸收，但不能在尚未验证前把具体网络层数写死。

Flowform 目标：

```text
Training-quality Model
        ↓ optional distillation
Runtime Quality Tier 0
Runtime Quality Tier 1
Runtime Quality Tier 2
```

Quality Tier 应由模型包声明，Runtime 决定当前角色使用哪个 Tier，以及多久 Evaluate 一次。

## 10. Flowform Lab 对架构的要求

Lab 不应该只画一个角色。

需要长期支持：

- Current Pose；
- Generated Pose；
- Next Pose / Future Chunk；
- Desired Trajectory；
- Predicted Trajectory；
- Blended / Extended Trajectory；
- Control Set；
- Root；
- Contact；
- Model State；
- Runtime timing；
- LOD / evaluation period；
- Scenario replay；
- Reference / Model compare。

UI 默认展示产品指标，高级面板再展示模型细节。

## 11. Engine Boundary

正式依赖方向：

```text
Game / Engine
     ↓
Engine Bridge
     ↓
Flowform Runtime API
     ↓
Model Package
```

Flowform Runtime 不知道 CharacterMovementComponent、Unity Playable 或 Godot Skeleton3D 的具体类型。

Unreal 是第一个正式支持引擎，不是核心 API 的宿主。

## 12. Third-party / Clean Implementation Boundary

- AnimGen：属于 Unreal Engine Experimental Plugin，可深入研究和测试，但 Flowform 跨引擎核心不能直接复制 Engine Code。
- Control Operators Reference：用于理解算法与最小实现；在许可未确认前不直接复制代码进入 Flowform。
- CAMDM：Unity 部分与其他源码许可证不同，代码、模型、数据分别处理。
- 所有参考项目均优先“复现 → 测量 → 抽象 → 独立实现”，不把第三方代码直接变成核心依赖。

详细许可状态见 `docs/licenses/third_party_registry.csv`。

## 13. 当前 Architecture Decision

截至 2026-08-17：

1. **接受** Control Schema 作为一级产品概念。
2. **接受** Pose AutoEncoder / Latent Motion 进入 M4 优先实验。
3. **接受** Runtime 同时支持 Autoregressive 与 Future-chunk 两类 Strategy。
4. **取消** “M4 必须使用 Small Transformer”的硬性约束。
5. **暂不决定** Flow Matching 是否成为 v1.0 唯一 Base Model。
6. **暂不决定** AutoEncoder 是否对所有模型包强制。
7. **保留** CAMDM / ARDY Future Buffer 路线作为正式可插拔 Backend 方向。
8. **保留** Flowform 的核心竞争力在框架、训练、调试、跨引擎与用户自定义，而不是单一网络创新。
