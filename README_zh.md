<div align="center">

#  DexBench

**灵巧手操作仿真评测基准**  
*覆盖强化学习、模仿学习与VLA方法，以触觉感知为核心对比维度*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Isaac Lab](https://img.shields.io/badge/仿真-Isaac%20Lab-green.svg)](https://isaac-sim.github.io/IsaacLab/)
[![MuJoCo](https://img.shields.io/badge/仿真-MuJoCo-orange.svg)](https://mujoco.org/)

[**论文**](#) | [**文档**](#) | [**English**](README.md)

</div>

---

## 目录

- [概述](#概述)
- [任务集](#任务集)
  - [第一梯度 — 基础任务](#第一梯度--基础任务触觉可选)
  - [第二梯度 — 接触敏感任务](#第二梯度--接触敏感任务推荐触觉)
  - [第三梯度 — 触觉关键任务](#第三梯度--触觉关键任务触觉必须)
  - [第四梯度 — 泛化与组合任务](#第四梯度--泛化与组合任务)
- [方法库](#方法库)
  - [强化学习方法](#强化学习方法)
  - [模仿学习方法](#模仿学习方法)
  - [视觉-语言-动作方法](#视觉-语言-动作方法)
- [方法 × 任务覆盖矩阵](#方法--任务覆盖矩阵)
- [评测指标](#评测指标)
- [支持的仿真环境](#支持的仿真环境)
- [支持的灵巧手模型](#支持的灵巧手模型)
- [安装](#安装)
- [快速开始](#快速开始)
- [仓库结构](#仓库结构)
- [路线图](#路线图)
- [引用](#引用)
- [开源协议](#开源协议)
- [致谢](#致谢)

---

## 概述

**DexBench** 是一个面向灵巧手操作策略的开源仿真评测基准，围绕三个在现有基准中被系统性忽视的核心维度构建：

1. **跨手型泛化** —— 同一策略在具有不同关节拓扑的多种灵巧手上进行评测（如四指欠驱动手、五指全驱动手、混合并联手）
2. **触觉感知作为受控变量** —— 每个接触密集型任务都包含"有触觉"与"无触觉"的配对评测，量化触觉传感器的边际价值
3. **方法多样性** —— 强化学习（RL）、模仿学习（IL）和视觉-语言-动作模型（VLA）三类方法在统一接口下均被支持

DexBench 设计上与实体灵巧手硬件配套使用。仿真栈针对 sim-to-real 迁移进行优化，并提供关节级动力学误差校准工具。

---

## 任务集

任务按接触敏感程度和时间跨度分为四个难度梯度。

### 第一梯度 —— 基础任务（触觉可选）

| 编号 | 任务名称 | 描述 | 触觉依赖程度 |
|------|----------|------|-------------|
| T01 | 多物体抓取 | 从杂乱场景中抓取目标物体，包含规则与不规则几何形状 | 低 |
| T02 | 手内重定向 | 单手将物体从初始姿态旋转到目标姿态 | 低–中 |
| T03 | 物体传递 | 双手间递交，或从手递交到固定夹具 | 中 |

### 第二梯度 —— 接触敏感任务（推荐触觉）

| 编号 | 任务名称 | 描述 | 触觉依赖程度 |
|------|----------|------|-------------|
| T04 | 精密插接 | 将连接器（USB-A/B、圆柱销、方形销）插入间隙为 0.5–2 mm 的插座 | **高** |
| T05 | 拧紧瓶盖/螺母 | 拧紧瓶盖和螺母，需感知是否达到目标扭矩 | **高** |
| T06 | 柔性物体操作 | 折叠布料、弯曲软管，需控制施力上限 | **高** |

### 第三梯度 —— 触觉关键任务（触觉必须）

| 编号 | 任务名称 | 描述 | 触觉依赖程度 |
|------|----------|------|-------------|
| T07 | 滑动检测与补救 | 抓取过程中检测即将滑落的趋势，并及时施加补救力 | **极高** |
| T08 | 盲插（视觉遮挡） | 手完全遮挡目标时完成插接，仅依赖触觉和本体感知 | **极高** |

### 第四梯度 —— 泛化与组合任务

| 编号 | 任务名称 | 描述 | 触觉依赖程度 |
|------|----------|------|-------------|
| T09 | 跨手型迁移 | 将 T01 策略在 3 种以上不同关节拓扑的灵巧手上进行评测 | 低–中 |
| T10 | 语言条件抓取 | 开放词汇指令跟随（如"把蓝色圆柱放入右边的槽"） | 低 |
| T11 | 长时程串联 | 顺序执行 3–5 个子任务：抓取 → 插接 → 拧紧 | **高** |

---

## 方法库

DexBench 提供三类范式共 17 种方法的参考实现，所有方法共享统一的观测/动作接口。

### 强化学习方法

| 编号 | 方法名称 | 核心思路 | 触觉 | 适用任务 |
|------|----------|----------|------|---------|
| [RL-01](methods/rl/RL-01.md) | PPO 稀疏奖励基线 | 标准 PPO，稀疏二元奖励 | ✗ | T01–T03 |
| [RL-02](methods/rl/RL-02.md) | PPO + ADR | 自动域随机化，提升跨手型鲁棒性 | ✗ | T01, T09 |
| [RL-03](methods/rl/RL-03.md) | RMA 在线适配 | 特权信息教师网络 → 本体历史学生网络蒸馏，实现在线 sim-to-real 适配 | ✗ | T02, T09 |
| [RL-04](methods/rl/RL-04.md) | 触觉奖励塑形 RL | LLM 自动生成触觉相关奖励函数（Text2Touch 风格）+ PPO | ✓ | T04, T05, T07 |
| [RL-05](methods/rl/RL-05.md) | 基础控制器 + 残差 RL | 先 RL 预训练低层原语（旋转/平移/捏合），再用残差 RL 组合高层任务 | ✗/✓ | T02, T04, T11 |
| [RL-06](methods/rl/RL-06.md) | DexNDM 关节校准 | 每种灵巧手用少量真实数据拟合关节级神经动力学误差模型，插入任何 sim-to-real pipeline | ✗ | T09 |

### 模仿学习方法

| 编号 | 方法名称 | 核心思路 | 触觉 | 适用任务 |
|------|----------|----------|------|---------|
| [IL-01](methods/il/IL-01.md) | Diffusion Policy 基线 | 标准去噪扩散策略，RGB + 关节状态输入 | ✗ | T01–T03 |
| [IL-02](methods/il/IL-02.md) | 反应式扩散策略 | 慢速扩散做 action chunk 预测 + 快速触觉 token 闭环做高频修正 | ✓ | T04, T05, T07 |
| [IL-03](methods/il/IL-03.md) | 流匹配策略 | 一致性流训练实现 1–2 步推理，适合高频接触控制任务 | ✗/✓ | T04, T11 |
| [IL-04](methods/il/IL-04.md) | FAAS 跨手型 BC | 功能-驱动器对齐空间统一不同手型关节表示，单一策略运行于多种手型 | ✗ | T09 |
| [IL-05](methods/il/IL-05.md) | DexMimicGen + BC | 从少量人工示范自动合成大量仿真示范，在扩增数据上训练 BC | ✗ | T01, T03, T11 |
| [IL-06](methods/il/IL-06.md) | 运动学锚定触觉 BC | 将触觉特征绑定到手部正运动学坐标系后送入策略观测层 | ✓ | T04, T08 |

### 视觉-语言-动作方法

| 编号 | 方法名称 | 核心思路 | 触觉 | 适用任务 |
|------|----------|----------|------|---------|
| [VLA-01](methods/vla/VLA-01.md) | OpenVLA-OFT 基线 | 开源 VLA + LoRA 微调；语言条件抓取基线 | ✗ | T01, T10 |
| [VLA-02](methods/vla/VLA-02.md) | VLM 规划器 + 扩散执行器 | VLM（Qwen2-VL）将任务分解为关键点/子目标，扩散策略负责执行 | ✗ | T10, T11 |
| [VLA-03](methods/vla/VLA-03.md) | VLA + ForceVLA-MoE | 在 VLA 动作解码头前插入力/触觉 MoE 路由模块 | ✓ | T04, T05, T07 |
| [VLA-04](methods/vla/VLA-04.md) | VLA-Touch（零侵入增强） | 触觉-语言模型生成语义触觉描述送入 VLM 规划器；扩散控制器做在线修正，不重训底层 VLA | ✓ | T04, T07 |
| [VLA-05](methods/vla/VLA-05.md) | 层级式 VLA | VLM 做语义分解和任务中途指令修正，低层策略执行；参考 Hi Robot / π0.5 架构 | ✗ | T10, T11 |

---

## 方法 × 任务覆盖矩阵

|  | T01 | T02 | T03 | T04 | T05 | T06 | T07 | T08 | T09 | T10 | T11 |
|--|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| RL-01 | ✓ | ✓ | ✓ | | | | | | | | |
| RL-02 | ✓ | | | | | | | | ✓ | | |
| RL-03 | | ✓ | | | | | | | ✓ | | |
| RL-04 | | | | ✓ | ✓ | | ✓ | | | | |
| RL-05 | | ✓ | | ✓ | | | | | | | ✓ |
| RL-06 | | | | | | | | | ✓ | | |
| IL-01 | ✓ | ✓ | ✓ | | | | | | | | |
| IL-02 | | | | ✓ | ✓ | | ✓ | | | | |
| IL-03 | | | | ✓ | | | | | | | ✓ |
| IL-04 | ✓ | | | | | | | | ✓ | | |
| IL-05 | ✓ | | ✓ | | | | | | | | ✓ |
| IL-06 | | | | ✓ | | | | ✓ | | | |
| VLA-01 | ✓ | | | | | | | | | ✓ | |
| VLA-02 | ✓ | | | | | | | | | ✓ | ✓ |
| VLA-03 | | | | ✓ | ✓ | | ✓ | | | | |
| VLA-04 | | | | ✓ | | | ✓ | | | | |
| VLA-05 | | | | | | | | | | ✓ | ✓ |

---

## 评测指标

除成功率外，DexBench 要求对每个任务强制报告以下指标：

| 指标 | 说明 |
|------|------|
| **成功率（SR）** | Episode 级别的二元成功判断 |
| **接触稳定性** | 每个 episode 中力超限次数 |
| **跨手型性能衰减率** | 策略迁移到未见手型时的成功率下降幅度 |
| **少样本数据效率** | 成功率随示范数量变化的曲线（IL/VLA 方法） |
| **阶段完成分数** | 长时程任务中子任务阶段的完成比例 |
| **触觉增益** | SR（有触觉）− SR（无触觉），量化触觉传感的边际价值 |

---

## 支持的仿真环境

| 环境 | 主要用途 | 说明 |
|------|---------|------|
| **Isaac Lab** | RL 训练、大规模并行 rollout、跨手型实验 | GPU 加速；支持 ADR、域随机化 |
| **MuJoCo / MuJoCo Playground** | 接触敏感任务（T04、T06、T08） | 接触动力学更精确；推荐用于精密插接任务 |

---

## 支持的灵巧手模型

| 手型 | 自由度 | 拓扑结构 | 触觉支持 |
|------|--------|---------|---------|
| Inspire RH56 | 12 | 五指腱驱动 | 可选（uSkin） |
| Allegro Hand v4 | 16 | 四指全驱动 | 可选（DIGIT） |
| Shadow Dexterous Hand | 24 | 五指、20 条独立腱 | 可选（BioTac） |
| *(自定义 URDF)* | — | 任意 | — |

---

## 安装

```bash
# 克隆仓库
git clone https://github.com/your-org/dexbench.git
cd dexbench

# 创建 conda 环境
conda create -n dexbench python=3.10
conda activate dexbench

# 先按官方文档安装 Isaac Lab
# https://isaac-sim.github.io/IsaacLab/

# 安装 DexBench
pip install -e .

# （可选）安装 MuJoCo 后端
pip install mujoco mujoco-mjx
```

---

## 快速开始

```bash
# 运行 RL 基线实验（T01 任务，Allegro 手，无触觉）
python train.py --task T01 --method RL-01 --hand allegro --sim isaaclab

# 精密插接任务有触觉 vs 无触觉对比
python train.py --task T04 --method IL-02 --hand inspire --tactile true
python train.py --task T04 --method IL-01 --hand inspire --tactile false

# 跨手型迁移评测（在 Allegro 上训练，在 Shadow + Inspire 上测试）
python train.py --task T01 --method IL-04 --hand allegro
python eval.py  --task T01 --method IL-04 --hand shadow,inspire --checkpoint path/to/ckpt
```

---

## 仓库结构

```
dexbench/
├── tasks/                  # 任务定义（奖励、重置、终止条件）
│   ├── tier1/              # T01–T03
│   ├── tier2/              # T04–T06
│   ├── tier3/              # T07–T08
│   └── tier4/              # T09–T11
├── methods/                # 方法实现
│   ├── rl/                 # RL-01 ~ RL-06
│   ├── il/                 # IL-01 ~ IL-06
│   └── vla/                # VLA-01 ~ VLA-05
├── envs/                   # 仿真环境封装
│   ├── isaac_lab/
│   └── mujoco/
├── hands/                  # 灵巧手 URDF/MJCF 资产和配置
├── tactile/                # 触觉传感器仿真与编码模块
├── eval/                   # 评测脚本与指标记录
├── configs/                # 实验配置文件（YAML）
├── scripts/                # 训练与评测入口脚本
├── docs/                   # 文档
├── README.md
└── README_zh.md
```

---

## 路线图

- [x] 任务集定义（T01–T11）
- [x] 方法接口规范
- [ ] Isaac Lab 环境实现（T01–T03, T09）
- [ ] MuJoCo 环境实现（T04, T06, T08）
- [ ] RL 基线实现（RL-01, RL-02）
- [ ] Diffusion Policy 基线（IL-01）
- [ ] 触觉传感器仿真模块
- [ ] 跨手型评测 pipeline
- [ ] VLA 集成（VLA-01, VLA-02）
- [ ] 完整排行榜与结果汇总表
- [ ] 真机 sim-to-real 迁移指南

---

## 引用

如果您在研究中使用了 DexBench，请引用：

```bibtex
@misc{dexbench2025,
  title   = {DexBench: A Simulation Benchmark for Dexterous Hand Manipulation},
  author  = {Your Name and Collaborators},
  year    = {2025},
  url     = {https://github.com/your-org/dexbench}
}
```

---

## 开源协议

本项目采用 MIT 协议，详见 [LICENSE](LICENSE)。

---

## 致谢

DexBench 在以下开源项目和研究工作的基础上构建或从中获得启发：Isaac Lab、MuJoCo Playground、OpenVLA-OFT、Diffusion Policy、CrossDex、DexMimicGen、ForceVLA、Reactive Diffusion Policy、DexterityGen 以及 DexNDM。感谢所有作者公开代码与数据。
