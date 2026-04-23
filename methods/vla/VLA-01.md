# VLA-01：OpenVLA-OFT 基线

**类型：** 视觉-语言-动作模型 | **触觉支持：** ✗ | **适用任务：** T01, T10

---

## 原始工作

- OpenVLA 论文：[OpenVLA: An Open-Source Vision-Language-Action Model](https://arxiv.org/abs/2406.09246)（Kim et al., 2024）
- OpenVLA 代码：[openvla/openvla](https://github.com/openvla/openvla)
- OpenVLA-OFT 论文：[Fine-Tuning Vision-Language-Action Models: Optimizing Speed and Success](https://arxiv.org/abs/2502.19645)（Pertsch et al., 2025）
- OpenVLA-OFT 代码：[moojink/openvla-oft](https://github.com/moojink/openvla-oft)

---

## 核心思路

**OpenVLA：** 基于 Prismatic VLM（7B 参数），将动作预测头接入 LLM 解码器，从自然语言指令 + 图像直接生成离散化动作 token。

**OFT（Optimized Fine-Tuning）关键改进：**
- **LoRA 微调：** 参数高效地将通用 VLA 适配到目标任务域
- **并行解码：** 同时预测多个动作 token，减少自回归推理步骤，大幅降低延迟
- **连续动作头：** 用回归头替代离散 token，提升精度

**语言条件：** 输入自然语言指令（如 "grasp the red cup"），VLM 负责语义理解和视觉定位。

---

## 在 DexBench 中的适配

| 设置 | 说明 |
|------|------|
| 仿真环境 | Isaac Lab |
| 基座模型 | OpenVLA-7B（Prismatic VLM）|
| 微调数据 | DexBench 遥操作示范 / DexMimicGen 合成数据 |
| 适用任务 | T01（语言条件抓取基线）、T10（开放词汇指令跟随）|

作为 VLA 方法的基准线，VLA-02 至 VLA-05 的性能均相对于此基线进行比较。

---

## 参考资料

- Kim, M.J., et al. (2024). *OpenVLA: An Open-Source Vision-Language-Action Model*. arXiv:2406.09246.
- Pertsch, K., et al. (2025). *Fine-Tuning Vision-Language-Action Models: Optimizing Speed and Success*. arXiv:2502.19645.
