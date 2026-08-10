# M1｜工作站与参考系统基线

> 项目：**流形 Flowform**  
> 阶段周期：**2026-08-10 ～ 2026-09-09**  
> 阶段目标：建立可重复实验的 Neural Motion 开发工作站，并完成参考系统基线。

## 1. M1 核心目标

M1 不以“学习神经网络”为目标，而是建立后续可以反复运行、训练、测试和收集结果的本地工作站。

最终工作流应达到：

```text
Flowform 工作站
        │
        ├── 开发环境固定
        ├── ARDY 跑通
        ├── MotionBricks 跑通
        ├── Kimodo 跑通
        ├── 数据 / 模型 / 日志目录规范
        └── 一键运行 + 一键打包报告
```

理想协作流程：

```text
定义实验
   ↓
本地一键运行
   ↓
人工体验 / 观察
   ↓
保存日志、截图、录像和配置
   ↓
打包 TestReport
   ↓
分析结果
   ↓
进入下一轮实验
```

## 2. 本阶段明确不做

M1 不做：

- Flowform 自有 Transformer 设计；
- Flowform Model V0 训练；
- Flowform Lab 正式网页开发；
- UE / Unity / Godot Bridge；
- C++ Runtime；
- TensorRT 性能优化；
- Style Training；
- 大规模动作数据收集；
- 系统性 PyTorch / 深度学习教学；
- 对 ARDY、MotionBricks、Kimodo 做深度二次开发。

原则：

> **先建立实验能力，再建立产品。**

M2 才进入 Flowform Lab，M3 建立数据体系，M4 才训练 Flowform 自有模型。

## 3. M1 工作站结构

### Windows 主机

```text
Windows 11
├─ NVIDIA Driver
├─ Git
├─ Git LFS
├─ VS Code
├─ Visual Studio / Build Tools
├─ CMake
├─ PowerShell
└─ Browser
```

Windows 主要承担：

- Git 与文件管理；
- Web 前端；
- 后续 C++ / Engine Bridge；
- 启停脚本；
- 查看实验结果。

### WSL2 / Ubuntu

```text
WSL2 / Ubuntu
├─ Python
├─ CUDA Runtime
├─ PyTorch
├─ CMake
├─ Ninja
├─ Git
└─ Python virtual environments
```

M1 原则：

> **AI / Training 优先在 WSL2 中运行。**

避免 Windows Python 与 WSL Python 混用导致环境不可复现。

---

# Week 1｜2026-08-10 ～ 2026-08-16
## 工作站、仓库和基础工具链

目标：让工作站成为“可以复现的开发环境”，而不是当前电脑上恰好能跑。

### W1-1 仓库基础结构

目标结构：

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
├─ workspace/      # gitignore
└─ reports/        # gitignore
```

M1 重点使用：

```text
tools/
tests/
scripts/
configs/
docs/
third_party/
workspace/
reports/
```

`third_party/`：第三方参考项目及其说明。  
`workspace/`：模型、Dataset、Cache、Checkpoint 等本地大文件，不进入 Git。  
`reports/`：实验报告，不默认进入 Git。

### W1-2 环境检查工具

建立第一版：

```text
flowform doctor
```

M1 可以先由以下脚本实现：

```text
scripts/doctor.ps1
scripts/doctor.sh
```

检查：

- OS；
- WSL；
- Ubuntu；
- Python；
- Git；
- Git LFS；
- GPU Name；
- VRAM；
- NVIDIA Driver；
- CUDA availability；
- PyTorch；
- `torch.cuda.is_available()`；
- CMake；
- Disk Free Space；
- RAM；
- Workspace 路径。

目标输出：

```text
Flowform Environment

Windows             PASS
WSL2                PASS
Ubuntu              PASS

GPU                  PASS
VRAM                 PASS
PyTorch              PASS
CUDA                 PASS
CUDA Device          PASS

Git                  PASS
Git LFS              PASS
CMake                PASS
Workspace            PASS

Environment Status
READY
```

环境状态必须自动检查，不能依赖“记得已经装过”。

### Week 1 验收

- [ ] Flowform 仓库建立；
- [ ] WSL2 可用；
- [ ] Python 环境可用；
- [ ] PyTorch 可运行；
- [ ] GPU 被 PyTorch 正确识别；
- [ ] CUDA inference / training tensor test 成功；
- [ ] Git LFS 正常；
- [ ] CMake 正常；
- [ ] Doctor 脚本可运行。

### Gate 1A

```text
Environment: READY
```

未通过不进入参考系统 Bring-up。

---

# Week 2｜2026-08-17 ～ 2026-08-23
## ARDY Baseline

本周是 M1 的核心参考系统验证。

重点观察：

```text
Target Velocity
Target Heading
Prediction Horizon
History
Auto-Replan
Replan Buffer
```

这些概念后续会映射到 Flowform 自己的 Motion Controller。

## W2-1 官方原版 Bring-up

原则：

> **先跑官方原版，不先改代码。**

流程：

```text
Repository Clone
        ↓
Dependencies
        ↓
Model / Dataset
        ↓
Official Demo
        ↓
GPU Inference
        ↓
Character Motion
```

首要问题是确认官方版本能否在 Flowform 工作站上可靠运行，而不是立即研究或重构内部实现。

## W2-2 ARDY Reference Card

建立：

```text
docs/reference/ARDY.md
```

只记录产品与架构层信息：

### 输入
- Velocity；
- Heading；
- History；
- Trajectory。

### 输出
- Pose；
- Root；
- Future Motion。

### Runtime 行为
- Prediction；
- Replan；
- Buffer。

### 可调参数
- Target Velocity；
- Target Heading；
- Prediction Horizon；
- Auto-Replan；
- 其他关键控制项。

文档目标不是论文综述，而是回答：

> **ARDY 如何被玩家实时控制？**

## W2-3 Motion Test Protocol V0

固定 8 个长期测试场景。

### A01｜Idle
无输入 10 秒。

检查：
- Pose Stability；
- Root Drift；
- Foot Drift。

### A02｜Idle → Walk
静止 → 轻输入 → 保持。

观察：
- 起步时间；
- 脚步；
- 身体朝向。

### A03｜Idle → Run
静止 → 强输入。

### A04｜Run → Stop

观察：

```text
Input ↓
Anchor 收近
Prediction 改变
Character Stop
```

### A05｜Stop → Recover

```text
Run
↓
释放输入
↓
角色开始准备 Stop
↓
突然重新输入
```

重点观察：
- Anchor 是否重新拉远；
- 是否触发 Replan；
- 旧 Stop 是否被取消；
- 是否重新进入 Run。

### A06｜90° Turn

### A07｜180° Turn

### A08｜Speed Change

```text
Walk
→ Run
→ Walk
```

这 8 个 Scenario 后续持续用于 Flowform Model V0 / V1、Benchmark 和 Engine Bridge 回归测试。

## W2-4 实验记录

每次实验至少记录：

- Experiment ID；
- Date；
- System；
- Commit；
- Model；
- Configuration；
- GPU；
- FPS；
- Inference Time；
- Notes；
- Result。

示例：

```text
EXP-ARDY-001

System: ARDY
Commit: <sha>
Test: A05 Stop-Recover
Result: PASS

Notes:
重新输入后角色明显重新规划；
停步已经开始但可以被中断。
```

### Week 2 验收

- [ ] ARDY 原版成功运行；
- [ ] GPU 推理成功；
- [ ] 官方 Demo 成功；
- [ ] A01～A08 全部执行；
- [ ] 关键 ARDY 参数可修改；
- [ ] Stop-Recover 行为观察完成；
- [ ] 实验结果可保存。

### Gate 1B

```text
ARDY: PASS
```

---

# Week 3｜2026-08-24 ～ 2026-08-30
## MotionBricks + Kimodo

本周不是学习两套完整系统，而是分别回答两个产品问题。

## MotionBricks：动作数据如何组织？

重点确认：

- Motion Representation；
- Dataset；
- Synthetic Training；
- VQVAE / Pose / Root 结构。

整理：

```text
输入数据
   ↓
Representation
   ↓
Training Dataset
   ↓
Model
   ↓
Pose / Root Output
```

### MotionBricks 验收

- [ ] 官方环境成功创建；
- [ ] Demo 或 Training Example 成功运行；
- [ ] 一个 Dataset 可以成功读取；
- [ ] 一个 Motion Clip 可以可视化；
- [ ] Skeleton 数据可以识别；
- [ ] Pose / Root 表示方式记录；
- [ ] 至少一次最小训练或推理流程完成。

重点不是训练质量，只要求：

```text
Training Pipeline: PASS
```

## Kimodo：未来如何低成本增加动作数据？

重点确认：

- Motion Generation；
- Constraint；
- Trajectory；
- Synthetic Dataset Generation。

主要问题：

```text
输入什么条件
↓
生成什么动作
↓
Trajectory 是否可控
↓
是否可批量生成
↓
输出能否进入未来 Flowform Dataset
```

### Kimodo 固定测试

- Walk Straight；
- Run Straight；
- 90° Turn；
- 180° Turn；
- Stop；
- Curve Trajectory。

观察：

- Trajectory Following；
- Motion Continuity；
- Foot Quality；
- Root Motion；
- Skeleton；
- Output Format。

最终判断它是否适合作为未来 Teacher / Synthetic Data Generator。

### Week 3 验收

```text
ARDY             PASS
MotionBricks     PASS
Kimodo           PASS
```

建立三份短技术卡：

```text
docs/reference/ARDY.md
docs/reference/MotionBricks.md
docs/reference/Kimodo.md
```

每份以 2～5 页等价信息量为目标，不做论文综述。

---

# Week 4｜2026-08-31 ～ 2026-09-09
## 自动化、实验体系和正式基线

前三周解决“能跑”，第四周解决：

> **一个月以后是否还能可靠、可重复地跑。**

## W4-1 环境锁定

记录：

- Python Version；
- PyTorch Version；
- CUDA；
- NVIDIA Driver；
- CMake；
- Compiler；
- Dependency Versions。

形成可复现的环境配置与 Version Manifest。

原则：

> 即使重装系统，也可以按文档重新建立 Flowform 工作站。

## W4-2 统一 Launcher

M1 不做复杂 GUI，以 PowerShell / BAT / Shell 为主。

目标入口：

```text
Flowform Launcher

1  Doctor
2  Run ARDY
3  Run MotionBricks
4  Run Kimodo
5  Package Report
```

目标脚本：

```text
scripts/doctor.ps1
scripts/run_ardy.ps1
scripts/run_motionbricks.ps1
scripts/run_kimodo.ps1
scripts/package_report.ps1
```

WSL 对应提供 `.sh` 版本或由 PowerShell 统一调用 WSL。

## W4-3 实验目录标准

每次实验生成：

```text
reports/
└─ 2026-09-03_ARDY_A05/
   ├─ environment.json
   ├─ config.json
   ├─ runtime.log
   ├─ metrics.json
   ├─ notes.txt
   ├─ screenshots/
   └─ video/
```

## W4-4 TestReport 打包

目标：

```text
package_report
```

得到：

```text
TestReport_ARDY_A05_20260903.zip
```

报告必须尽量包含足以复现实验的环境、代码版本、模型、配置与人工观察记录。

## W4-5 Reference / License Registry

建立第三方登记表，代码、模型与数据许可证分开记录。

第一批至少包含：

| 项目 | 用途 | 代码许可 | 模型许可 | 数据许可 | 是否可分发 |
|---|---|---|---|---|---|
| ARDY | Runtime Reference | 待核验 | 待核验 | 待核验 | 待核验 |
| MotionBricks | Training Reference | 待核验 | 待核验 | 待核验 | 待核验 |
| Kimodo | Data / Generation Reference | 待核验 | 待核验 | 待核验 | 待核验 |

在实际审查前不对具体许可证做推断。

---

# 4. Sprint 划分

M1 进一步拆为约 2～4 天一个 Sprint：

| Sprint | 内容 | 完成标准 |
|---|---|---|
| S1 | 工作站审计 + 仓库 + WSL2 | 基础环境可用 |
| S2 | CUDA / PyTorch + Doctor | Environment READY |
| S3 | ARDY Bring-up | 官方 Demo GPU 运行 |
| S4 | ARDY 测试体系 | A01～A08 完成 |
| S5 | MotionBricks | Training Pipeline PASS |
| S6 | Kimodo | Generation Pipeline PASS |
| S7 | Launcher / Report | 一键运行与打包 |
| S8 | 冻结基线 + M1 验收 | Gate A PASS |

## 当前第一个 Sprint：M1-S1

当前优先任务：

1. 建立 Flowform 主仓库；
2. 检查 Windows 开发环境；
3. 检查 WSL2；
4. 检查 NVIDIA / CUDA 状态；
5. 建立 Python AI 环境；
6. PyTorch 成功识别本机 NVIDIA GPU；
7. 跑第一个 GPU Tensor Test；
8. 保存 EnvironmentReport。

在这一步完成前，暂不安装 ARDY、MotionBricks、Kimodo，避免三个参考项目分别带入不同 CUDA / Python / PyTorch 环境造成混乱。

### M1-S1 完成标准

> **得到一份可复现的工作站环境报告，并确认工作站具备后续 Flowform Neural Motion 实验条件。**

---

# 5. M1 职责划分

## 产品负责人 / 本地执行

主要负责：

- 安装程序；
- 执行命令 / 脚本；
- Windows GUI 操作；
- 下载大模型 / Dataset；
- 运行 Demo；
- 操作角色；
- 录像 / 截图；
- 动作体验判断；
- 提供错误日志与 TestReport。

优先承担低认知但高时间成本工作，例如下载、解压、点击、重复测试、录像、长时间训练和人工动作观察。

## 架构 / 工程侧

主要负责：

- 软件与版本方案；
- 仓库与目录设计；
- 环境脚本；
- Doctor；
- 安装错误分析；
- 参考代码审查；
- ARDY / MotionBricks / Kimodo 架构抽象；
- 测试协议；
- License 审核；
- 决定哪些技术值得进入 Flowform；
- 为 M2 Flowform Lab 确定接口抽象。

---

# 6. M1 正式成果

M1 结束时至少形成：

1. **Reproducible Workstation** — 环境可以重新建立；
2. **Environment Doctor** — 一键判断工作站状态；
3. **Reference Baseline** — ARDY / MotionBricks / Kimodo 全部运行；
4. **Motion Test Protocol V0** — A01～A08；
5. **Experiment Report System** — 运行 → 测试 → 保存 → 打包；
6. **Reference Architecture Notes** — 能回答三个参考系统各自提供什么价值；
7. **Third-party Registry** — 第三方代码 / 模型 / 数据许可分别登记。

# 7. M1 最终验收表

| 项目 | 要求 |
|---|---|
| Windows Dev Environment | PASS |
| WSL2 | PASS |
| GPU | PASS |
| CUDA | PASS |
| PyTorch GPU | PASS |
| Git / Git LFS | PASS |
| CMake | PASS |
| Environment Doctor | PASS |
| ARDY | **PASS** |
| MotionBricks | **PASS** |
| Kimodo | **PASS** |
| ARDY A01～A08 | PASS |
| Experiment Report | PASS |
| Auto Package | PASS |
| Repository Structure | PASS |
| Version Lock | PASS |
| Reference Notes | PASS |
| License Register | PASS |

## Gate A｜Reference Systems Ready

通过条件：

```text
一条统一入口启动 / 检查环境

ARDY             PASS
MotionBricks     PASS
Kimodo           PASS

GPU              PASS
CUDA             PASS

日志 / 实验报告可自动打包
```

Gate A 通过后正式进入：

> **M2 — Flowform Lab Alpha**
