# Flowform 参考系统矩阵

> 更新：2026-08-17  
> 目的：统一管理参考项目的价值、可复现边界、研究问题和进入 Flowform 的方式。

## 1. 原则

参考项目不是 Flowform 的依赖清单。

每个参考系统必须分别回答：

1. **它解决了什么问题？**
2. **哪些结果可以在我们机器上复现？**
3. **哪些只是预训练 Demo / 能力展示？**
4. **代码、模型、数据是否允许复用 / 再分发？**
5. **它改变的是 Flowform 哪一层，而不是“是否全盘换路线”？**

统一研究流程：

```text
Source Verification
      ↓
License Boundary
      ↓
Official Baseline
      ↓
Reproduction / Smoke Test
      ↓
Fixed Scenario Benchmark
      ↓
Architecture Evidence Card
      ↓
Flowform Decision
```

## 2. 优先级总表

| Priority | Reference | 可复现性 | Flowform 价值 | 当前处理方式 |
|---|---|---|---|---|
| **S** | Unreal Engine AnimGen | Player：高；NPC 高级训练：不可复现 | Control Schema、AutoEncoder、Flow Matching、编辑器训练、LOD / Distillation | M1 首要官方 Baseline |
| **S** | Control Operators Reference | 简化实现可训练；并非论文 / UE 原始完整实现 | 可组合 Control、Flow-Matching Autoregressive Controller 的最小透明实现 | M1 完整跑通 Python Reference |
| **A** | ARDY | 依可获得仓库 / 模型而定 | Player Intent、Trajectory、Anchor、Prediction、Replan | M1 Control / Replan Baseline |
| **A** | CAMDM | PyTorch 训练 + Unity 推理公开 | Future Chunk、Lazy Replan、Trajectory Fusion、Diffusion、Style、Benchmark | M1 最小 Bring-up；M5 深入 |
| **A** | MotionBricks | 依当前可获得实现而定 | Motion Representation、模块化网络、异步 Runtime、高性能 | M1 架构 / Bring-up 对照 |
| **B** | Kimodo | 依当前可获得模型 / 数据而定 | Motion Generation、Teacher、Synthetic Dataset | M3 / M6 数据方向研究 |

## 3. AnimGen / Control Operators

### 我们能确认的部分

Epic UE5.8 AnimGen API 将 `UAnimGenController` 定义为：使用 Flow Matching 与神经网络、自回归逐帧生成角色 Pose 的训练后 Controller；Controller 与一个 Behavior / Control Structure 关联。

AnimGen 官方示例包含两套不同性质的 Controller：

#### Player Controller

- Basic walking locomotion；
- Aim；
- Fire pistol；
- 官方示例提供训练数据；
- 可以在 Sample 中重新训练。

因此它是 **可复现训练 Baseline**。

#### NPC Controller

- Walking locomotion；
- 大量 Style；
- Prop 支持；
- 官方不提供训练数据；
- 只能使用预训练 Controller 观察 Runtime 与规模化能力。

因此它是 **Capability Reference，不是 Reproducible Training Baseline**。

### Control Operators Python Reference

论文 / 作者公开了简化参考实现，用于解释可组合 Control Operators 和 Flow-Matching Autoregressive Controller。

需要牢记：作者明确说明，因为原始代码和数据无法公开，公开 Python 项目是额外准备的**小型示例实现**，不能当作 UE AnimGen / 论文完整生产实现的源代码替代品。

### Flowform 吸收内容

- Composable Control Schema；
- Control Encoder 与 Model 分离；
- Pose AutoEncoder / Latent Representation；
- Autoregressive frame-by-frame Runtime Strategy；
- Reference Model → Distillation → Runtime Quality Tier / LOD 的产品思想；
- 训练和调试需要成为设计师可操作工具。

### Flowform 不直接吸收

- Unreal Engine 类型和 Engine Code；
- 未公开 NPC 训练数据 / pipeline 的猜测；
- 把 Flow Matching 写死为 Flowform 唯一模型。

详见 [`ANIMGEN_BASELINE.md`](ANIMGEN_BASELINE.md)。

## 4. CAMDM

CAMDM 是另一条非常重要但不同的实时生成路线：

```text
Past Motion
+
Future Trajectory / Orientation
+
Style
      ↓
Conditional Diffusion
      ↓
Future Motion Chunk
      ↓
Buffer / Lazy Replan
```

对 Flowform 的核心价值：

- 一次生成未来多帧；
- Future Motion Buffer；
- Lazy Replan；
- Predicted Trajectory 与 User Intent 分离；
- Future Trajectory Extension / Fusion；
- Control Trajectory Augmentation；
- Style Transition；
- Research Benchmark。

因此 CAMDM 不与 AnimGen 二选一：二者分别代表 Flowform Runtime 必须支持的两类模型策略。

详见 [`CAMDM.md`](CAMDM.md)。

## 5. ARDY

ARDY 的研究问题保持：

```text
玩家输入如何变成未来运动意图？
Prediction Horizon 如何影响响应？
Stop / Recover 如何触发重新规划？
Anchor / Trajectory 如何变化？
```

固定重点 Scenario：

- Run → Stop；
- Stop → Recover；
- 90°；
- 180°；
- Rapid Input Change。

ARDY 的价值主要在 Runtime Control，而不是决定 Flowform 最终网络结构。

## 6. MotionBricks

当前把 MotionBricks 定义为架构级参考：

- Motion Representation；
- 多阶段 / 模块化模型；
- 大规模动画语料；
- 异步 Runtime；
- 推理线程与游戏线程解耦；
- 高性能游戏集成。

社交媒体 / 视频中的性能数字必须重新确认统计口径，不能把“Game Thread 提交开销”和“完整推理耗时”直接比较。

如果完整训练代码 / 数据不可获得，则 M1 不要求虚构“复现训练”；改为记录 Source、可获得组件、Runtime 证据和未公开边界。

## 7. Kimodo

Kimodo 暂不进入 Runtime 主路线。

主要评估：

- 是否可按 Trajectory / Constraint 生成动作；
- 是否适合批量补充 locomotion / transition；
- 输出是否可以进入 Flowform Canonical Dataset；
- 生成数据的许可证与来源能否满足未来开源 / 商用训练。

因此它是 M3 / M6 的数据来源候选，不阻塞 M1 Core Gate。

## 8. M1 统一记录字段

每个可运行参考都至少记录：

```text
Reference Name
Source URL
Version / Commit
Engine / Python Version
GPU / Driver / CUDA
Dataset / Model
Training Config
Training Duration
Peak VRAM
Output Artifacts
Runtime FPS / Frame Cost
Scenario Results
Screenshots / Video
Known Limitations
Reproducibility Level
License Status
Reviewer Notes
```

### Reproducibility Level

- `R3 FULL`：训练数据、训练代码、Runtime 都可重复执行；
- `R2 BASELINE`：核心训练 / Runtime 可重复，但生产级数据或部分实现缺失；
- `R1 RUNTIME`：只能运行预训练结果；
- `R0 EVIDENCE`：只能做论文 / 视频 / 架构分析。

当前预期：

```text
AnimGen Player             R3/R2（以实际训练结果确认）
AnimGen NPC                R1
Control Operators Ref      R2
CAMDM                      R3/R2
ARDY                       待实际 Bring-up
MotionBricks               待确认
Kimodo                     待确认
```

## 9. 主要来源

### AnimGen
- Epic Tutorial: https://dev.epicgames.com/community/learning/tutorials/eGME/unreal-engine-animgen
- Epic AnimGen API: https://dev.epicgames.com/documentation/unreal-engine/API/Plugins/AnimGen
- UAnimGenController: https://dev.epicgames.com/documentation/unreal-engine/API/Plugins/AnimGen/UAnimGenController

### Control Operators
- Publication overview: https://theorangeduck.com/page/control-operators-interactive-character-animation
- Publications index: https://theorangeduck.com/page/publications
- Reference code: https://github.com/gouruiyu/ControlOperators

### CAMDM
- Repository: https://github.com/AIGAnimation/CAMDM
- Paper: https://arxiv.org/abs/2404.15121

## 10. 当前结论

Flowform 不押注“哪一个 Demo 最强”。

当前最有价值的结论是：

```text
AnimGen / Control Operators
证明可组合 Control + Autoregressive Flow Controller 是成熟方向

CAMDM / ARDY
证明 Future Plan + Buffer + Replan 也是成熟方向

Flowform
应该统一这些模型，而不是被其中任何一个模型定义
```
