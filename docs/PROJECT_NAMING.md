# Flowform 项目命名规范

## 正式名称

- **中文名：流形**
- **英文名：Flowform**
- **技术定位：Neural Motion Framework**
- **品牌归属：A BAIGE Project**

推荐对外标准写法：

> **流形 Flowform**  
> Neural Motion Framework  
> *A BAIGE Project*

## 名称含义

**Flowform = Flow + Form**。

表达“流动中的形态 / 形态随运动连续变化”，对应 Flowform 的核心方向：通过神经运动模型与实时控制系统生成连续、可响应的游戏角色运动。

## 核心组件命名

正式产品组件统一使用：

```text
Flowform Lab
Flowform Training
Flowform Runtime
Flowform Model

Flowform UE
Flowform Unity
Flowform Godot
```

### Flowform Lab
Web 端运动验证与研发工具。

主要承担 Runtime Debug、Dataset Inspector、Model Compare、Training Monitor、Style Preview、Benchmark 等功能。

### Flowform Training
训练与数据工具链。

主要承担 Import、Retarget、Preprocess、Train / Fine-tune、Benchmark、Export。

### Flowform Runtime
跨引擎、Engine-independent C++ Runtime。

负责实时推理、Motion Buffer、Root / Pose / Contact / Replan 等核心运行时逻辑。

### Flowform Model
Flowform 自有模型及其版本体系。

例如：

```text
Flowform Model V0
Flowform Model V1
```

### Engine Bridges

```text
Flowform UE
Flowform Unity
Flowform Godot
```

Bridge 仅负责把各引擎的角色、Skeleton、输入和动画系统接入 Flowform Runtime，不把核心 Runtime 逻辑锁死在某个引擎内。

## 品牌层级

```text
BAIGE / 百舸
│
└─ Flowform / 流形
   Neural Motion Framework
   │
   ├─ Flowform Lab
   ├─ Flowform Training
   ├─ Flowform Runtime
   ├─ Flowform Model
   └─ Engine Bridges
      ├─ Flowform UE
      ├─ Flowform Unity
      └─ Flowform Godot
```

## 仓库命名

当前正式主仓库：

```text
Syurli/Flowform
```

仓库名直接使用产品英文名，不再使用旧工程代号。

## 旧命名处理

历史讨论中出现的：

```text
NeuralMotion
Neural Motion
NeuralMotionRuntime
NeuralMotion V0
```

在正式产品代码和新文档中逐步替换为：

```text
Flowform
Flowform Runtime
Flowform Model V0
```

但以下情况可以保留 `Neural Motion`：

1. 作为技术领域描述，例如 “neural motion generation”；
2. 引用论文、第三方项目或其原始术语；
3. `Neural Motion Framework` 作为 Flowform 的技术定位描述。

原则：

> **Flowform 是产品名；Neural Motion 是技术类别。**

## 代码命名建议

新建 Flowform 自有代码时优先使用 `Flowform` 前缀。

示例：

```text
FlowformRuntime
FlowformModel
FlowformSkeleton
FlowformMotionIntent
FlowformTrajectoryAnchor
FlowformProvider
```

引擎层示例：

```text
UFlowformComponent
UFlowformModel
UFlowformSkeletonMap
FAnimNode_Flowform
```

这些名称属于当前架构方向，具体类名在进入对应实现阶段时可以再冻结，但禁止重新引入 `NeuralMotion` 作为正式产品前缀。
