# VLA-05：层级式 VLA

**类型：** 视觉-语言-动作模型 | **触觉支持：** ✗ | **适用任务：** T10, T11

---

## 原始工作

- π0.5 论文：[π0.5: a Vision-Language-Action Model with Open-World Generalization](https://arxiv.org/abs/2504.16054)（Black et al. / Physical Intelligence, 2025）
- π0 代码：[physical-intelligence/openpi](https://github.com/physical-intelligence/openpi)
- Hi Robot：*TODO — 补充 Hi Robot 论文链接*
- Qwen3-VL 代码：[QwenLM/Qwen3](https://github.com/QwenLM/Qwen3)

---


![pai0.5](pai0.5.png)

## 核心思路

**层级结构：**

| 层次 | 模型 | 职责 | 运行频率 |
|------|------|------|---------|
| 高层（VLM）| Qwen3-VL | 语义分解、任务监控、中途指令修正 | 极低频（关键事件触发）|
| 低层（策略）| 动作模型（见下表）| 执行具体动作序列 | 高频（10–50 Hz）|

**与 VLA-02（固定规划-执行分离）的区别：**
- VLA-02：VLM 在任务开始时规划，执行期间不再介入
- VLA-05：VLM 持续监控执行状态，检测到子任务完成或异常时**动态介入**重新规划

**π0.5 / Hi Robot 特性：**
- VLM 作为"语义大脑"持续关注执行进度
- 支持开放域语言指令修正（"先把红色的拿走，然后把蓝色的放进去"）
- 低层策略接受 VLM 的动态条件输入，无需针对每条指令重训

---

## 可选低层策略

| 执行器 | 推理步数 | 多模态建模 | 实机验证 | 适合场景 |
|--------|---------|-----------|---------|---------|
| Diffusion Policy（IL-01）| 10–20 步 | ✓✓ | ✓ | 精度优先 |
| Flow Matching（IL-03）| 1–2 步 | ✓✓ | ✓ | 延迟敏感、高频控制 |
| ACT | 1 步 | ✓ | ✓✓ | 真机部署、双手操作 |
| Consistency Policy | 1 步 | ✓✓ | △ | 速度与精度折中 |
| Transformer BC | 1 步 | ✗ | ✓ | 下界基线对比 |

**推荐组合：**
- T10（开放词汇）：Qwen3-VL + Flow Matching（VLM 已占主要推理时间，执行器需快速响应）
- T11（长时程）：Qwen3-VL + ACT（子任务切换频繁，ACT 的动作分块机制适合段落式执行）

---

## 在 DexBench 中的适配

| 设置 | 说明 |
|------|------|
| 仿真环境 | Isaac Lab |
| 高层 VLM | Qwen3-VL |
| 低层策略 | 见上表，默认使用 Flow Matching（IL-03）|
| 适用任务 | T10（开放词汇，强调语义理解）、T11（长时程，强调动态任务调度）|
| 对照实验 | 与 VLA-02 对比：固定规划 vs. 动态介入的长时程任务性能差异；同时对比不同低层策略 |

---

## 参考资料

- Black, K., et al. (2025). *π0.5: a Vision-Language-Action Model with Open-World Generalization*. arXiv:2504.16054.
- Zhao, T., et al. (2023). *Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware*. arXiv:2304.13705.（ACT）
- Chi, C., et al. (2023). *Diffusion Policy: Visuomotor Policy Learning via Action Diffusion*. arXiv:2303.04137.
- Black, K., et al. (2024). *π0: A Vision-Language-Action Flow Model for General Robot Control*. arXiv:2410.24164.
- Prasad, A., et al. (2024). *Consistency Policy: Accelerated Visuomotor Policies via Consistency Distillation*. arXiv:2405.07503.
