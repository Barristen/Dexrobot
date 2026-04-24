# VLA-02：VLM + 动作模型

**类型：** 视觉-语言-动作模型 | **触觉支持：** ✗ | **适用任务：** T10, T11

---

## 架构图
![组合](star.png)


---

## 原始工作

- Qwen3-VL 代码：[QwenLM/Qwen3](https://github.com/QwenLM/Qwen3)
- Diffusion Policy：[real-stanford/diffusion_policy](https://github.com/real-stanford/diffusion_policy)（Chi et al., 2023）
- Flow Matching：[π0](https://github.com/physical-intelligence/openpi)（Black et al., 2024）
- ACT：[tonyzhaozh/act](https://github.com/tonyzhaozh/act)（Zhao et al., 2023）
- Consistency Policy：[Consistency Policy](https://arxiv.org/abs/2405.07503)（Prasad et al., 2024）

---

## 核心思路

**两层解耦架构：**

| 层次 | 模型 | 输入 | 输出 | 运行频率 |
|------|------|------|------|---------|
| 语义规划层 | Qwen3-VL | 图像 + 语言指令 | 子目标关键点 / 子任务描述 | 低频（每子任务触发一次）|
| 执行层 | 动作模型（见下表）| 图像 + 关节状态 + 子目标条件 | 动作序列 | 高频（10–50 Hz）|

**VLM 规划器职责：**
1. 解析自然语言指令，分解为有序子任务
2. 在图像中定位目标物体（视觉 grounding）
3. 生成子任务的关键帧条件（传递给执行器）
4. 监控执行进度，触发下一子任务

**与 VLA-01 的区别：** VLA-01 是端到端 VLA；VLA-02 将语义规划与低层控制显式分离，执行器可独立替换与优化。

---

## 可选执行器

| 执行器 | 推理步数 | 多模态建模 | 实机验证 | 适合场景 |
|--------|---------|-----------|---------|---------|
| Diffusion Policy（IL-01）| 10–20 步 | ✓✓ | ✓ | 精度优先 |
| Flow Matching（IL-03）| 1–2 步 | ✓✓ | ✓ | 延迟敏感、高频控制 |
| ACT | 1 步 | ✓ | ✓✓ | 真机部署、双手操作 |
| Consistency Policy | 1 步 | ✓✓ | △ | 速度与精度折中 |
| Transformer BC | 1 步 | ✗ | ✓ | 下界基线对比 |

**推荐组合：**
- T10（开放词汇）：Qwen3-VL + Flow Matching（低延迟，VLM 已占主要计算时间）
- T11（长时程）：Qwen3-VL + ACT（实机验证充分，子任务切换稳定）

---

## 在 DexBench 中的适配

| 设置 | 说明 |
|------|------|
| 仿真环境 | Isaac Lab |
| 规划器 | Qwen3-VL（zero-shot 或 few-shot）|
| 执行器 | 见上表，默认使用 Diffusion Policy 作为基准 |
| 适用任务 | T10（开放词汇，VLM 理解多样指令）、T11（长时程，VLM 管理子任务切换）|
| 消融实验 | 固定 Qwen3-VL 规划器，对比不同执行器的性能与推理延迟 |

---

## 参考资料

- Zhao, T., et al. (2023). *Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware*. arXiv:2304.13705.（ACT）
- Chi, C., et al. (2023). *Diffusion Policy: Visuomotor Policy Learning via Action Diffusion*. arXiv:2303.04137.
- Black, K., et al. (2024). *π0: A Vision-Language-Action Flow Model for General Robot Control*. arXiv:2410.24164.
- Prasad, A., et al. (2024). *Consistency Policy: Accelerated Visuomotor Policies via Consistency Distillation*. arXiv:2405.07503.
