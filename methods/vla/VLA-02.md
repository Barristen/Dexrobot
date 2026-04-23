# VLA-02：VLM 规划器 + 扩散执行器

**类型：** 视觉-语言-动作模型 | **触觉支持：** ✗ | **适用任务：** T10, T11

---

## 原始工作

- Qwen2-VL 论文：[Qwen2-VL: Enhancing Vision-Language Model's Perception of the World at Any Resolution](https://arxiv.org/abs/2409.12191)（Wang et al., 2024）
- Qwen2-VL 代码：[QwenLM/Qwen2-VL](https://github.com/QwenLM/Qwen2-VL)
- 扩散执行器：基于 IL-01（Diffusion Policy），见 [real-stanford/diffusion_policy](https://github.com/real-stanford/diffusion_policy)

---

## 核心思路

**两层解耦架构：**

| 层次 | 模型 | 输入 | 输出 | 运行频率 |
|------|------|------|------|---------|
| 语义规划层 | Qwen2-VL | 图像 + 语言指令 | 子目标关键点 / 子任务描述 | 低频（每子任务触发一次）|
| 执行层 | Diffusion Policy | 图像 + 关节状态 + 子目标 | 动作序列 | 高频（~10 Hz）|

**VLM 规划器职责：**
1. 解析自然语言指令，分解为有序子任务
2. 在图像中定位目标物体（视觉 grounding）
3. 生成子任务的关键帧条件（传递给扩散执行器）
4. 监控执行进度，触发下一子任务

**扩散执行器职责：** 以子目标为条件，生成低层动作序列；不需要理解高层语义。

**与 VLA-01 的区别：** VLA-01 是端到端 VLA；VLA-02 将语义规划与低层控制显式分离，便于分别优化。

---

## 在 DexBench 中的适配

| 设置 | 说明 |
|------|------|
| 仿真环境 | Isaac Lab |
| 规划器 | Qwen2-VL-7B（zero-shot 或 few-shot）|
| 执行器 | 在 DexBench 数据上微调的 Diffusion Policy |
| 适用任务 | T10（开放词汇，VLM 理解多样指令）、T11（长时程，VLM 管理子任务切换）|

---

## 参考资料

- Wang, P., et al. (2024). *Qwen2-VL: Enhancing Vision-Language Model's Perception of the World at Any Resolution*. arXiv:2409.12191.
