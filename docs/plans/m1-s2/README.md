# M1-S2 — AnimGen Official Sample Baseline

> 目标：完整走通 UE5.8 AnimGen 官方样板的“运行 → Player 重新训练 → 本地产物重新运行 → 对比记录”链路。

## 任务边界

本任务只做官方 Baseline，不改网络结构，不删训练数据，不替换 Control，不重构 Sample。

必须区分：

```text
Player Controller
- 官方提供训练数据
- 可重新训练
- 本任务必须重训一次

NPC Controller
- 官方不提供训练数据
- 不能在 Sample 中重训
- 只运行、观察、记录
```

## Step 1｜获取与打开样板

- 获取 Epic 官方 AnimGen Sample；
- 使用匹配版本的 UE5.8；
- 确认 Experimental AnimGen Plugin 正常；
- 记录 Sample / Engine 版本；
- 不改 Sample Content。

输出：

```text
reports/animgen-baseline-<date>/environment.json
reports/animgen-baseline-<date>/sample_version.txt
```

Gate：`ANIMGEN_SAMPLE_OPEN=PASS`

## Step 2｜预训练 Player Runtime

执行并录像：

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

记录：FPS、可获得的 Runtime Timing、LOD、Evaluation Period、Foot Slide、Root、响应感。

Gate：`ANIMGEN_PLAYER_PRETRAINED=PASS`

## Step 3｜预训练 NPC Capability Record

记录：

- Style 规模与切换；
- Props；
- Behavior / Control 组织；
- LOD / Evaluation Period；
- 多角色能力（若样板直接支持）；
- Runtime 成本；
- 资产 / 模型结构。

报告必须写：

```text
Training Data: NOT PROVIDED
Training Reproduction: NOT AVAILABLE
Reproducibility: R1 RUNTIME
```

Gate：`ANIMGEN_NPC_CAPABILITY_RECORD=PASS`

## Step 4｜按官方默认配置重训 Player

- 记录 AnimDatabase / Training Data；
- 记录 AutoEncoder 配置；
- 记录 Controller 配置；
- 使用官方默认路径训练；
- 保存训练日志 / TensorBoard 信息；
- 记录开始结束时间；
- 记录 Peak VRAM / RAM；
- 保存训练产物位置与大小；
- 不为了追求结果擅自改参数。

Gate：`ANIMGEN_PLAYER_RETRAIN=PASS`

## Step 5｜本地训练产物 Runtime

用本地训练的 Player 重跑 A01～A11。

生成对比表：

```text
Epic Pre-trained
vs
Local Re-trained
```

至少比较：Start、Stop、Stop-Recover、90 / 180、Rapid Change、Aim、Fire、Foot Slide、Runtime Cost。

Gate：`ANIMGEN_LOCAL_RUNTIME=PASS`

## Step 6｜输出 Baseline Report

仓库提交一份小型摘要，不提交大型模型和视频。

建议摘要路径：

```text
docs/reference/results/ANIMGEN_PLAYER_BASELINE_<date>.md
```

报告回答：

1. 官方流程是否完整跑通？
2. 训练耗时？
3. 峰值显存？
4. 训练产物大小？
5. 本地结果与官方预训练差异？
6. 最明显的动作优缺点？
7. Control / Behavior 如何映射到 Flowform Control Schema？
8. 哪些能力只在 NPC Demo 中看到但无法训练验证？

## Final Gate

```text
Sample Open                  PASS
Player Pretrained            PASS
NPC Capability Record        PASS
Player Re-train              PASS
Local Re-trained Runtime     PASS
Baseline Report              PASS

M1-S2 = PASS
Next = M1-S3 Control Operators Reference
```

## Reference

完整研究说明：`docs/reference/ANIMGEN_BASELINE.md`
