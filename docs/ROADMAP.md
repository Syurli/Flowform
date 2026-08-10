# Flowform 一年开发路线图

> 项目：**流形 Flowform**  
> 技术定位：**Neural Motion Framework**  
> 品牌归属：**A BAIGE Project**  
> 周期：**2026-08-10 ～ 2027-08-09**

## 1. 年度产品目标

第一年不以研究论文或单一 Demo 为终点，而是交付一个可以实际开源使用的：

> **跨引擎、可训练的 Neural Locomotion Framework**

目标用户为独立开发者和中小型游戏团队。

理想用户流程：

```text
安装 Flowform
    ↓
导入自己的 Locomotion 动画
BVH / FBX / Mocap
    ↓
Skeleton Mapping / Retarget
    ↓
自动数据预处理
    ↓
基于 Base Model 训练 / 微调自己的 Style
    ↓
Flowform Lab 网页实时试玩和调参
    ↓
Export Model Package
    ↓
Unreal / Unity / Godot
    ↓
实时 Neural Locomotion
```

## 2. v1.0 范围

第一代重点覆盖：

- Locomotion；
- Start / Stop；
- 加减速；
- 任意方向转向；
- Strafe / Backward；
- Continuous Transition；
- Style-conditioned Locomotion；
- 基础姿态状态切换；
- 为未来 Traversal / Interaction 留接口。

第一年**不追求神经网络替代所有动画系统**。攻击、换弹、技能、处决以及严格 Timing 的动作仍允许传统 Animation / Montage 等系统接管。

Traversal、Smart Object、Hand Interaction、Full Body Target Pose、Combat Transition 等能力可以在第一年研究接口，但正式能力放在 v1.x / v2.0。

## 3. 产品架构

```text
                         Flowform

 ┌──────────────────────────────────────────────┐
 │                  Flowform Lab                │
 │                                              │
 │ Runtime Debug / Dataset Inspector            │
 │ Model Compare / Training Monitor             │
 │ Style Preview / Benchmark                    │
 └──────────────────────┬───────────────────────┘
                        │
 ┌──────────────────────▼───────────────────────┐
 │               Flowform Training              │
 │                                              │
 │ Import → Retarget → Process → Train          │
 │ Fine-tune → Benchmark → Export               │
 └──────────────────────┬───────────────────────┘
                        │
                 Flowform Model Package
                     .onnx + metadata
                        │
 ┌──────────────────────▼───────────────────────┐
 │                Flowform Runtime              │
 │                                              │
 │ Engine-independent C++ Runtime               │
 │ Motion Buffer / Async Inference              │
 │ Root / Pose / Contact / Replan               │
 └───────────────┬───────────────┬──────────────┘
                 │               │
           Flowform UE     Flowform Unity
                 │
           Flowform Godot
```

核心约束：

> **Flowform Training 与引擎通过 Canonical Skeleton + Model Package 解耦。**

不能训练出只能供某一个引擎使用的模型。

## 4. 12 个月里程碑

| 月份 | 阶段 | 核心交付 | 状态目标 |
|---|---|---|---|
| M1 | 工作站与参考系统 | ARDY / MotionBricks / Kimodo + 项目骨架 | 能完整实验 |
| M2 | Flowform Lab Alpha | 网页实时操控 ARDY | 第一套产品原型 |
| M3 | 数据系统 | Canonical Skeleton + Dataset Pipeline | 摆脱特定仓库格式 |
| M4 | **Flowform Model V0** | 自研小型 Transformer | **网页运行自己的模型** |
| M5 | Motion Controller V1 | Anchor / Replan / Buffer / Root | 操控达到可玩 |
| M6 | 数据与质量体系 | Benchmark + Teacher Dataset | 可系统迭代 |
| M7 | Style Training | Style Condition / Adapter | 用户可训练风格 |
| M8 | Flowform Runtime Alpha | ONNX + C++ Runtime | 脱离 Python Runtime |
| M9 | Flowform UE Alpha | Unreal Locomotion 实装 | 第一个游戏引擎 |
| M10 | 跨引擎 Beta | Unity + Godot Bridge | 架构验证完成 |
| M11 | Open Source Beta | Training UX / 示例 / CI / 文档 | 外部开发者可使用 |
| M12 | **v1.0** | 开源发布 + Base Model + Samples | 第一代产品完成 |

---

## M1｜工作站与参考系统基线
**2026-08-10 ～ 2026-09-09**

目标：建立可重复运行、训练、测试和收集报告的本地工作站。

主要内容：

- Windows 开发环境；
- WSL2 / Ubuntu AI 环境；
- CUDA / PyTorch / Python / CMake / Git LFS；
- 验证 ARDY；
- 验证 MotionBricks；
- 验证 Kimodo；
- 建立 Environment Doctor；
- 建立实验记录与 TestReport 打包；
- 建立第三方依赖 / License Register。

M1 Gate：

```text
Environment      PASS
GPU / CUDA       PASS
ARDY             PASS
MotionBricks     PASS
Kimodo           PASS
Report Packaging PASS
```

详细计划：[`plans/M1_WORKSTATION_BASELINE.md`](plans/M1_WORKSTATION_BASELINE.md)

---

## M2｜Flowform Lab Alpha
**2026-09-10 ～ 2026-10-09**

开始制作第一个属于 Flowform 的产品原型。

初步技术链：

```text
Babylon.js + TypeScript
          ↕
       WebSocket
          ↕
FastAPI / Python Motion Server
          ↕
        ARDY Provider
```

Flowform Lab 展示：

- Current Pose；
- Generated Pose；
- Trajectory Anchor；
- Desired / Generated Trajectory；
- Root；
- Foot Contacts；
- Current / Target Speed；
- Heading；
- Prediction Horizon；
- Inference Time；
- Buffer Size；
- Replan Count。

建立核心抽象：

```text
IMotionModel

Predict(
    History,
    MotionIntent
)
→ FutureMotion
```

ARDY 只作为 `ArdyProvider`，后续 Flowform Model 可以直接替换。

验收：浏览器完成 WASD / Controller 的 Idle、Walk、Run、Stop、Turn、Re-accelerate，并能观察玩家意图 → Anchor → Prediction → Runtime Motion。

**Gate B：Flowform Lab Alpha。**

---

## M3｜Flowform 数据体系
**2026-10-10 ～ 2026-11-09**

建立自己的 Canonical Skeleton 与统一动作数据格式。

第一版 Canonical Skeleton 约 22～30 个主要骨骼，不处理 Face / Finger。

核心数据结构：

```text
MotionFrame
MotionClip
SkeletonDefinition
MotionDataset
MotionIntent
TrajectoryAnchor
FootContact
```

特征至少支持：

- Joint Local Rotation6D；
- Joint Position；
- Joint Velocity；
- Root Position；
- Root Velocity；
- Root Heading；
- Root Angular Velocity；
- Foot Contact。

数据管线：

```text
BVH / NPZ / CSV
       ↓
Skeleton Mapping
       ↓
Retarget
       ↓
Resample
       ↓
Normalize
       ↓
Motion Feature
       ↓
Sliding Window
       ↓
Training Dataset
```

Flowform Lab 增加 Dataset Inspector。

目标 CLI：

```text
flowform import ./motions/
flowform preprocess
flowform inspect
```

**Gate C：训练数据不再依赖 ARDY / MotionBricks 内部格式。**

---

## M4｜Flowform Model V0
**2026-11-10 ～ 2026-12-09**

训练第一代自有模型。

V0 不做 Diffusion、VQ-VAE、Motion Token 或文本控制，先验证最小闭环：

```text
Past Motion
+
Player Intent
+
Trajectory Anchor
       ↓
Small Transformer
       ↓
Future Motion
```

初步模型规模：5M～20M parameters。

重点场景：

- Idle → Walk；
- Idle → Run；
- Run → Stop；
- Run → Stop → Run；
- Forward → Left / Right；
- 90° Turn；
- 180° Turn；
- Slow → Fast；
- Fast → Slow。

重点重现：

```text
输入减弱
→ Anchor 收近
→ 预测停步
→ 正在减速

输入重新恢复
→ Anchor 拉远
→ Replan
→ 中断停步
→ 重新加速
```

验收：Flowform Lab 可在 ARDY 与 Flowform Model V0 之间切换，自有模型独立实时运行。动作质量暂不要求优于 ARDY，要求技术闭环属于 Flowform。

**Gate D：Own Model Milestone。**

---

## M5｜Motion Controller V1
**2026-12-10 ～ 2027-01-09**

从“会预测动作”进入“游戏 Locomotion Controller”。

```text
Motion History
Trajectory Anchor
Motion Intent
        ↓
Neural Motion Planner
        ↓
Future Motion Buffer
        ↓
Runtime
```

Runtime 加入：

- Prediction Buffer；
- History Buffer；
- Replan Buffer；
- Generation ID；
- Interpolation；
- Root Correction；
- Foot Lock；
- Contact。

Root Motion 与 Body Motion 明确拆分。

核心 KPI：Input Response、Stopping Distance、Turning Responsiveness、Foot Sliding、Pose Jitter、Root Error。

验收：动作开始具有游戏操控感，而不只是“看起来像人在运动”。

---

## M6｜Benchmark 与数据生成体系
**2027-01-10 ～ 2027-02-09**

固定长期回归 Scenario：

```text
Start
Stop
Stop-Recover
90°
180°
Strafe
Backward
Acceleration
Deceleration
Rapid Input Change
```

每个模型自动生成：

```text
TestReport/
├─ metrics.json
├─ input.json
├─ motion.npz
├─ video/
├─ screenshots/
└─ runtime.log
```

同时建立 Teacher Data Pipeline，在许可允许的前提下利用外部模型 / 数据源补充动作数据。

目标：模型版本 V0.x 可以自动、可重复比较。

---

## M7｜Style Training
**2027-02-10 ～ 2027-03-09**

产品差异化目标：**Bring Your Own Motion Style**。

典型 Style：Normal、Military、Heavy、Robot、Injured、Zombie、Crouched 等。

研究两条路线：

- Style Embedding；
- Lightweight Style Adapter。

目标不是让用户重训完整模型，而是：

> 少量自己的动作 → 得到自己的运动风格。

Training Toolkit 产品化目标：

```text
flowform doctor
flowform import
flowform preprocess
flowform train-style
flowform benchmark
flowform export
```

验收：Base Style + 至少两个明显不同的 Style，在同一输入下肉眼可区分。

**Gate E：Custom Style Milestone。**

---

## M8｜Flowform Runtime Alpha
**2027-03-10 ～ 2027-04-09**

建立 Engine-independent C++17/20 Runtime。

核心 API 方向：

```cpp
Runtime.PushPose(...);
Runtime.SetMotionIntent(...);
Runtime.SetTrajectoryAnchor(...);
Runtime.Update(...);
Runtime.GetPose(...);
```

Backend：

- ONNX Runtime CPU；
- ONNX Runtime CUDA；
- TensorRT。

第一代模型包建议：

```text
MyModel.flowform/
├─ model.onnx
├─ skeleton.json
├─ normalization.json
├─ model.json
├─ style.json
└─ license.json
```

> 模型包最终扩展名 / 容器格式在实现阶段冻结，此处仅表示内容结构。

验收：Flowform Lab 可以切换 Python Runtime 与 C++ Runtime，输出在容差范围内一致。

**Gate F：Runtime Independence。**

---

## M9｜Flowform UE Alpha
**2027-04-10 ～ 2027-05-09**

第一个正式游戏引擎落地。

目标结构：

```text
CharacterMovement
       ↓
Flowform Component
       ↓
Flowform Runtime
       ↓
Motion Buffer
       ↓
AnimNode
       ↓
CompactPose
```

第一版功能：

- Load Model；
- Skeleton Mapping；
- Locomotion Input；
- Async Inference；
- Pose Output；
- Debug Draw；
- Fallback Animation。

验收：UE 标准第三人称 Demo 完成 Idle、Walk、Run、Stop、Turn、Strafe、Style。

**Gate G：第一个真正游戏版本。**

---

## M10｜跨引擎验证
**2027-05-10 ～ 2027-06-09**

验证 Flowform 是 Runtime，而不是 UE 专用插件。

Unity：Native Runtime + AnimationJob / Playable。  
Godot：GDExtension + Skeleton3D。

不要求功能与 UE 同等完整，目标是证明同一个 Model Package 可供：

```text
Flowform Lab
Flowform UE
Flowform Unity
Flowform Godot
```

使用并得到基本一致的 Locomotion。

**Gate H：Cross-engine Architecture Validated。**

---

## M11｜Open Source Beta
**2027-06-10 ～ 2027-07-09**

停止扩张大功能，验证“陌生开发者能否真的使用”。

交付：

- Installer / Environment Setup；
- Training Preset；
- Sample Dataset；
- Base Model；
- Model Export；
- Flowform UE Sample；
- Flowform Unity Sample；
- Flowform Godot Sample；
- Flowform Lab。

完善 `flowform doctor`，检查 GPU、CUDA、PyTorch、VRAM、Disk、Dependencies、Dataset、Model。

CI：

- Windows CI；
- Linux CI；
- Training Smoke Test；
- ONNX Export；
- Runtime Tests；
- Engine Bridge Builds / Tests。

同时完成 LICENSE、NOTICE、Third-party Attribution、Dataset License Matrix、Model Card。

---

## M12｜v1.0 Release
**2027-07-10 ～ 2027-08-09**

原则：**禁止继续研发新的大功能。**

重点：Bug Fix、Performance、Packaging、Documentation、Samples、Regression、Benchmark、Release。

正式交付：

### Flowform Runtime
- Windows / Linux；
- CPU / CUDA；
- TensorRT 正式路径。

### Flowform Lab
- Runtime Debug；
- Dataset Inspector；
- Model Benchmark；
- Style Preview。

### Flowform Training
- Import；
- Preprocess；
- Train / Fine-tune；
- Benchmark；
- Export。

### Engine Bridges
- Flowform UE：正式支持；
- Flowform Unity：Beta；
- Flowform Godot：Beta。

### Models
- Base Locomotion Model；
- 2～3 Style Examples。

### Documentation
- 5 Minute Quick Start；
- Train Your First Style；
- Unreal Integration；
- Unity Integration；
- Godot Integration；
- Model Format；
- Runtime API；
- Architecture。

---

## 5. 五个年度关键 Gate

### Gate 1｜2026 年 10 月
ARDY 驱动的 Flowform Lab 可以实时玩。  
**证明：产品框架成立。**

### Gate 2｜2026 年 12 月
ARDY 可以被 Flowform Model V0 替换。  
**证明：核心 AI 技术独立。**

### Gate 3｜2027 年 1 月
自有模型具有可接受的游戏操控响应。  
**证明：不只是 AI Demo。**

### Gate 4｜2027 年 3 月
用户能够训练自己的 Style。  
**证明：产品是工具，而不是固定模型。**

### Gate 5｜2027 年 6 月
同一个模型包运行在多个引擎。  
**证明：成为跨引擎 Locomotion Runtime。**

## 6. 年度成功标准

| 指标 | v1.0 标准 |
|---|---|
| 自有模型 | ✅ 不依赖 ARDY 推理 |
| 实时 Locomotion | ✅ |
| Start / Stop / Replan | ✅ |
| 方向 / 速度连续控制 | ✅ |
| Style Training | ✅ |
| 用户自定义动作训练 | ✅ |
| Flowform Lab | ✅ |
| Flowform Runtime | ✅ |
| ONNX | ✅ |
| Windows / Linux | ✅ |
| Flowform UE | ✅ 正式支持 |
| Flowform Unity | ✅ Beta |
| Flowform Godot | ✅ Beta |
| 自动训练环境 | ✅ |
| Model Package | ✅ |
| 开源文档 | ✅ |
| CI / Test | ✅ |

第一年最关键的技术时间点：

> **2026 年 12 月前，Flowform 自有模型必须能够在网页中实时运行。**

年度节奏：

> **前 4 个月证明 AI → 中间 3 个月证明产品 → 后 3 个月证明跨引擎 → 最后 2 个月完成开源产品化。**

## 7. 协作原则

项目采用产品与结果驱动协作：

- 产品负责人负责产品目标、动作表现判断、体验验收、本地执行与 UX 决策；
- 架构、模型结构、Loss、Dataset Schema、训练代码、Runtime、跨引擎 ABI、实验策略、性能分析和问题定位由工程侧承担；
- 不以“学习神经网络知识”作为里程碑；
- 只有直接影响产品决策的技术概念才需要进入产品层讨论；
- Loss 等指标是诊断工具，最终标准是游戏中的动作质量、稳定性与操控响应。
