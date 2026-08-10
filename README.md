# 流形 Flowform

**Neural Motion Framework**  
*A BAIGE Project*

Flowform（流形）是一个面向游戏角色运动的开源、可训练、跨引擎 Neural Locomotion Framework。

项目目标不是用神经网络替代所有游戏动画，而是优先解决实时 Locomotion、Start / Stop、加减速、任意方向转向、Strafe / Backward、Continuous Transition 与 Style-conditioned Locomotion，并为 Traversal、Interaction 等后续能力保留接口。

## Product Architecture

Flowform 由四层核心产品组成：

- **Flowform Lab** — Web 运动验证、Runtime Debug、Dataset Inspector、Model Compare、Training Monitor、Style Preview 与 Benchmark。
- **Flowform Training** — 动作导入、Skeleton Mapping / Retarget、预处理、训练 / 微调、Benchmark 与模型导出。
- **Flowform Runtime** — Engine-independent C++ Runtime，负责 Motion Buffer、异步推理、Root / Pose / Contact / Replan 等运行时能力。
- **Engine Bridges** — Flowform UE、Flowform Unity、Flowform Godot。

训练系统与引擎通过 **Canonical Skeleton + Model Package** 解耦，目标是让同一模型包能够跨引擎使用。

## v1.0 Target

计划周期：**2026-08-10 ～ 2027-08-09**。

第一年目标：交付一个可实际开源使用的跨引擎 Neural Locomotion Framework，包括：

- 自有实时 Locomotion 模型，不依赖参考系统推理；
- Start / Stop / Replan；
- 连续方向与速度控制；
- Style Training 与用户自定义动作训练；
- Flowform Lab Web；
- Flowform Training；
- Engine-independent C++ Runtime；
- ONNX Runtime / CUDA，并为 TensorRT 后端留出正式路径；
- Windows / Linux；
- Flowform UE 正式支持；
- Flowform Unity / Godot Beta；
- Model Package、自动化环境、测试、CI 与文档。

完整路线图见 [`docs/ROADMAP.md`](docs/ROADMAP.md)。  
当前 M1 执行计划见 [`docs/plans/M1_WORKSTATION_BASELINE.md`](docs/plans/M1_WORKSTATION_BASELINE.md)。

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
├─ third_party/
├─ workspace/      # local only
└─ reports/        # local only
```

## Current Phase

**M1 — Workstation & Reference Systems Baseline**  
2026-08-10 ～ 2026-09-09

当前阶段不训练 Flowform 自有模型。阶段目标是建立可重复实验的 Windows / WSL2 工作站，跑通 ARDY、MotionBricks、Kimodo，建立环境检测、实验记录和一键报告打包能力。

M1 Gate：

```text
Environment      PASS
GPU / CUDA       PASS
ARDY             PASS
MotionBricks     PASS
Kimodo           PASS
Report Packaging PASS
```

通过后进入 M2：**Flowform Lab Alpha**。

## Development Principle

Flowform 采用结果驱动的研发方式：

- 产品负责人负责产品目标、操控体验、动作表现判断与验收；
- 架构、模型、训练、Runtime、实验设计与问题定位以工程实现为中心推进；
- Loss 等训练指标用于诊断，但最终标准是游戏中的动作质量与操控响应。

## License

项目计划开源，但代码、参考项目、模型与训练数据的许可证需要分别管理。正式 License 将在依赖与第三方许可矩阵完成核验后确定。
