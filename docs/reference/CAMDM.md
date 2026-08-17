# CAMDM 参考卡

> Reference: Taming Diffusion Probabilistic Models for Character Control, SIGGRAPH 2024  
> Flowform Priority: **A**  
> Updated: 2026-08-17

## 1. 定位

CAMDM 对 Flowform 的价值不是“证明我们必须改用 Diffusion”，而是提供了一套完整的 Future-chunk 实时角色控制范式：

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
Runtime Buffer
      ↓
Consume / Lazy Replan
```

它与 AnimGen / Control Operators 的 frame-by-frame autoregressive 路线互补。

## 2. Flowform 重点吸收内容

### Future Motion Buffer

模型一次生成未来多帧，Runtime 连续消费其中一部分，再根据策略触发下一次生成。

Flowform 因此保留正式 `FutureChunk` Runtime Strategy，而不是把所有 Backend 都改造成 Next-Pose Controller。

### Lazy Replan

并非每一帧都重新生成完整未来动作。Replan interval 是质量、响应与成本之间的重要参数。

### Predicted vs Desired Trajectory

Runtime 应区分：

```text
Model Predicted Trajectory
User / AI Desired Trajectory
Blended Control Trajectory
Extended Prediction / Fusion State
```

这些调试通道未来应进入 Flowform Lab。

### Future Trajectory Extension / Fusion

当旧预测已经消费一部分，而新 Desired Trajectory 仍覆盖完整 Horizon 时，需要处理长度、连续性和下一轮条件跳变问题。

CAMDM 的 HFTE 思路值得在 M5 Future-chunk Backend 实验，但不直接写死成 Flowform 唯一算法。

### Control Trajectory Augmentation

训练时的未来控制不能永远等于 Mocap 的完美 Ground Truth Future。

Flowform M3 因此加入：

- trajectory smoothing；
- perturbation；
- noise；
- stop / recover；
- rapid direction change；
- imperfect player / AI intent。

### Style

CAMDM 展示一个统一模型控制大量 locomotion styles，并提供 Style Transition 相关研究参考。

Flowform M7 将其作为 Style Embedding / Transition Baseline，但最终目标仍然是用户少量动作 → 自定义 Style，而不是只支持固定离散标签。

## 3. Benchmark 参考

CAMDM Evaluation 对 Flowform M6 的研究指标有直接参考价值：

- motion quality；
- foot sliding；
- acceleration；
- trajectory alignment；
- orientation alignment；
- style accuracy；
- diversity；
- style transition。

Flowform 在此基础上增加游戏指标：

- input response；
- stop distance；
- recovery / replan latency；
- 90 / 180 turn；
- rapid input change；
- runtime frame cost；
- multi-character scaling。

## 4. M1 最小 Baseline

M1 不要求为了“证明跑过”而完整执行长时间全量 100STYLE 训练。

优先顺序：

1. [ ] 锁定官方仓库版本；
2. [ ] 跑通官方预训练 / Unity Runtime；
3. [ ] 记录 WASD、Style、Facing / Orientation 控制；
4. [ ] 跑固定 Scenario；
5. [ ] 观察 Future Buffer / Replan / Trajectory Debug；
6. [ ] 跑一次最小训练烟雾测试，确认 Dataset → Training → Checkpoint → ONNX 路径；
7. [ ] 如果时间允许，再安排独立的完整训练实验。

Gate 可以是：

```text
CAMDM_RUNTIME = PASS
CAMDM_TRAINING_PIPELINE = PASS or DOCUMENTED_DEFER
CAMDM_ARCHITECTURE_CARD = PASS
```

## 5. 与 AnimGen 的对照

| Dimension | AnimGen / Control Operators | CAMDM |
|---|---|---|
| Generation | Next Pose / frame-by-frame | Future Motion Chunk |
| Main Model | Flow Matching | Diffusion |
| Runtime State | autoregressive pose state | history + future buffer |
| Control Change | next generation step | replan / fusion |
| Long-horizon Preview | optional / indirect | natural |
| Runtime Complexity | autoregressive stability / LOD | buffer / replan / fusion |
| Flowform Role | V0-A primary experiment | FutureChunk backend reference |

Flowform 不做二选一，而是用统一 Runtime API 容纳两类策略。

## 6. License Boundary

官方 CAMDM 仓库 README 当前明确说明：

- Unity code：GPL-3；
- 其他 source code：Apache License 2.0。

代码许可不代表模型和训练数据自动具有相同许可。100STYLE、Mixamo / retargeted data、pretrained checkpoint 等必须分别登记。

Flowform 的处理方式：

```text
Paper / PyTorch / Unity Runtime
        ↓
Reference / Reproduction / Benchmark
        ↓
Independent Flowform Implementation
```

尤其不要把 GPL Unity Runtime 直接变成 Flowform 跨引擎核心。

## 7. Sources

- Repository: https://github.com/AIGAnimation/CAMDM
- Paper: https://arxiv.org/abs/2404.15121
