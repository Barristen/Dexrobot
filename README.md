<div align="center">

#  DexBench

**A Simulation Benchmark for Dexterous Hand Manipulation**  
*Covering RL, Imitation Learning, and VLA Methods with Tactile Sensing as a First-Class Dimension*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Isaac Lab](https://img.shields.io/badge/sim-Isaac%20Lab-green.svg)](https://isaac-sim.github.io/IsaacLab/)
[![MuJoCo](https://img.shields.io/badge/sim-MuJoCo-orange.svg)](https://mujoco.org/)

[**Paper**](#) | [**Documentation**](#) | [**中文版**](README_zh.md)

</div>

---

## Table of Contents

- [Overview](#overview)
- [Task Suite](#task-suite)
  - [Tier 1 — Foundation Tasks](#tier-1--foundation-tasks-tactile-optional)
  - [Tier 2 — Contact-Sensitive Tasks](#tier-2--contact-sensitive-tasks-tactile-recommended)
  - [Tier 3 — Tactile-Critical Tasks](#tier-3--tactile-critical-tasks-tactile-required)
  - [Tier 4 — Generalization & Composition](#tier-4--generalization--composition-tasks)
- [Method Library](#method-library)
  - [Reinforcement Learning](#reinforcement-learning)
  - [Imitation Learning](#imitation-learning)
  - [Vision-Language-Action](#vision-language-action)
- [Method × Task Coverage](#method--task-coverage)
- [Evaluation Metrics](#evaluation-metrics)
- [Supported Simulation Environments](#supported-simulation-environments)
- [Supported Hand Models](#supported-hand-models)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Repository Structure](#repository-structure)
- [Roadmap](#roadmap)
- [Citation](#citation)
- [License](#license)
- [Acknowledgements](#acknowledgements)

---

## Overview

**DexBench** is an open simulation benchmark for evaluating learning-based dexterous hand manipulation policies. It is designed around three core axes that are systematically underexplored in existing benchmarks:

1. **Cross-morphology generalization** — the same policy evaluated across hands with different topologies (e.g., 4-finger underactuated, 5-finger fully actuated, parallel-jaw hybrid)
2. **Tactile sensing as a controlled variable** — every contact-rich task includes paired evaluations with and without tactile/force feedback, quantifying the marginal value of tactile sensing
3. **Method diversity** — RL, Imitation Learning, and Vision-Language-Action (VLA) methods are all supported under a unified interface

DexBench is built to complement physical dexterous hand hardware. The simulation stack is designed for sim-to-real transfer, with calibration tools for joint-level dynamics gap correction.

---

## Task Suite

Tasks are organized into four difficulty tiers based on contact sensitivity and temporal horizon.

### Tier 1 — Foundational (Tactile Optional)

| ID | Task | Description | Tactile Dependency |
|----|------|-------------|-------------------|
| T01 | Multi-Object Grasping | Grasp target objects from cluttered scenes; includes regular and irregular geometries | Low |
| T02 | In-Hand Reorientation | Reorient an object from an initial to a goal pose within a single hand | Low–Medium |
| T03 | Object Handover | Transfer objects between hands or from hand to a fixed fixture | Medium |

### Tier 2 — Contact-Sensitive (Tactile Recommended)

| ID | Task | Description | Tactile Dependency |
|----|------|-------------|-------------------|
| T04 | Precision Insertion | Insert connectors (USB-A/B, cylindrical pins, square pegs) into sockets with 0.5–2 mm clearance | **High** |
| T05 | Screw and Cap Tightening | Tighten bottle caps and nuts; requires sensing when target torque is reached | **High** |
| T06 | Deformable Object Handling | Fold cloth, bend soft tubing; requires force upper-bound control | **High** |

### Tier 3 — Tactile-Critical (Tactile Required)

| ID | Task | Description | Tactile Dependency |
|----|------|-------------|-------------------|
| T07 | Slip Detection and Recovery | Detect incipient slip during grasp and apply corrective force in time | **Very High** |
| T08 | Blind Insertion | Complete insertion when the hand fully occludes the target; relies purely on tactile and proprioception | **Extreme** |

### Tier 4 — Generalization and Composition

| ID | Task | Description | Tactile Dependency |
|----|------|-------------|-------------------|
| T09 | Cross-Morphology Transfer | Evaluate T01 policies across 3+ hand types with different joint topologies | Low–Medium |
| T10 | Language-Conditioned Grasping | Open-vocabulary instruction following (e.g., "place the blue cylinder into the right slot") | Low |
| T11 | Long-Horizon Chaining | Sequential execution of 3–5 subtasks: grasp → insert → tighten | **High** |

---

## Method Library

DexBench provides reference implementations for 17 methods across three paradigms. All methods share a unified observation/action interface.

### Reinforcement Learning

| ID | Method | Key Idea | Tactile | Target Tasks |
|----|--------|----------|---------|-------------|
| [RL-01](methods/rl/RL-01.md) | PPO Sparse Baseline | Standard PPO with sparse binary reward | ✗ | T01–T03 |
| [RL-02](methods/rl/RL-02.md) | PPO + ADR | Automatic Domain Randomization for cross-morphology robustness | ✗ | T01, T09 |
| [RL-03](methods/rl/RL-03.md) | RMA Online Adaptation | Privileged teacher → proprioceptive-history student distillation for online sim-to-real adaptation | ✗ | T02, T09 |
| [RL-04](methods/rl/RL-04.md) | Tactile Reward Shaping | LLM-generated tactile reward functions (Text2Touch-style) + PPO | ✓ | T04, T05, T07 |
| [RL-05](methods/rl/RL-05.md) | Foundation Controller + Residual RL | Pre-train low-level primitives (rotate/translate/pinch) via RL; compose with residual RL for downstream tasks | ✗/✓ | T02, T04, T11 |
| [RL-06](methods/rl/RL-06.md) | DexNDM Joint Calibration | Per-hand joint-wise neural dynamics correction fitted from a small real dataset; plugs into any sim-to-real pipeline | ✗ | T09 |

### Imitation Learning

| ID | Method | Key Idea | Tactile | Target Tasks |
|----|--------|----------|---------|-------------|
| [IL-01](methods/il/IL-01.md) | Diffusion Policy Baseline | Standard denoising diffusion policy; RGB + joint state input | ✗ | T01–T03 |
| [IL-02](methods/il/IL-02.md) | Reactive Diffusion Policy | Slow diffusion for action chunk prediction + fast tactile-token loop for high-frequency correction | ✓ | T04, T05, T07 |
| [IL-03](methods/il/IL-03.md) | Flow Matching Policy | Consistency flow training for 1–2 step inference; suited for high-frequency contact tasks | ✗/✓ | T04, T11 |
| [IL-04](methods/il/IL-04.md) | FAAS Cross-Morphology BC | Function-Actuator-Aligned Space unifies joint representations across hand topologies; single policy runs on multiple hands | ✗ | T09 |
| [IL-05](methods/il/IL-05.md) | DexMimicGen + BC | Auto-synthesize large demonstration sets from a small number of human demos; train BC on augmented data | ✗ | T01, T03, T11 |
| [IL-06](methods/il/IL-06.md) | Kinematically-Anchored Tactile BC | Bind tactile features to hand FK coordinate frame before feeding into policy observation | ✓ | T04, T08 |

### Vision-Language-Action

| ID | Method | Key Idea | Tactile | Target Tasks |
|----|--------|----------|---------|-------------|
| [VLA-01](methods/vla/VLA-01.md) | OpenVLA-OFT Baseline | Open-source VLA with LoRA fine-tuning; language-conditioned grasping baseline | ✗ | T01, T10 |
| [VLA-02](methods/vla/VLA-02.md) | VLM Planner + Diffusion Executor | VLM (Qwen2-VL) decomposes task into keypoints/subgoals; diffusion policy executes | ✗ | T10, T11 |
| [VLA-03](methods/vla/VLA-03.md) | VLA + ForceVLA-MoE | Insert force/tactile MoE routing module before VLA action decoding head | ✓ | T04, T05, T07 |
| [VLA-04](methods/vla/VLA-04.md) | VLA-Touch (Non-invasive) | Tactile-language model generates semantic tactile descriptions → VLM planner; diffusion controller applies online corrections without retraining base VLA | ✓ | T04, T07 |
| [VLA-05](methods/vla/VLA-05.md) | Hierarchical VLA | VLM handles semantic decomposition and mid-task instruction correction; low-level policy executes; inspired by Hi Robot / π0.5 | ✗ | T10, T11 |

---

## Method × Task Coverage

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

## Evaluation Metrics

Beyond success rate, DexBench mandates reporting the following per task:

| Metric | Description |
|--------|-------------|
| **Success Rate (SR)** | Episode-level binary success |
| **Contact Stability** | Number of force-limit violations per episode |
| **Cross-Morphology Drop** | Performance delta when transferring policy to an unseen hand type |
| **Few-Shot Sample Efficiency** | SR as a function of number of demonstrations (IL/VLA methods) |
| **Phase Completion Score** | Fraction of subtask stages completed (long-horizon tasks) |
| **Tactile Gain** | SR(with tactile) − SR(without tactile); quantifies marginal value of tactile sensing |

---

## Supported Simulation Environments

| Environment | Primary Use | Notes |
|-------------|-------------|-------|
| **Isaac Lab** | RL training, large-scale parallel rollouts, cross-morphology experiments | GPU-accelerated; supports ADR, domain randomization |
| **MuJoCo / MuJoCo Playground** | Contact-sensitive tasks (T04, T06, T08) | More accurate contact dynamics; recommended for precision insertion |

---

## Supported Hand Models

| Hand | DoF | Topology | Tactile Support |
|------|-----|----------|----------------|
| Inspire RH56 | 12 | 5-finger tendon-driven | Optional (uSkin) |
| Allegro Hand v4 | 16 | 4-finger fully actuated | Optional (DIGIT) |
| Shadow Dexterous Hand | 24 | 5-finger, 20 independent tendons | Optional (BioTac) |
| *(Custom URDF)* | — | Any | — |

---

## Installation

```bash
# Clone the repository
git clone https://github.com/Barristen/dexbench.git
cd dexbench

# Create conda environment
conda create -n dexbench python=3.10
conda activate dexbench

# Install Isaac Lab (follow official instructions first)
# https://isaac-sim.github.io/IsaacLab/

# Install DexBench
pip install -e .

# (Optional) Install MuJoCo backend
pip install mujoco mujoco-mjx
```

---

## Quick Start

```bash
# Run a baseline RL experiment (T01, Allegro hand, no tactile)
python train.py --task T01 --method RL-01 --hand allegro --sim isaaclab

# Run tactile vs. no-tactile comparison on precision insertion
python train.py --task T04 --method IL-02 --hand inspire --tactile true
python train.py --task T04 --method IL-01 --hand inspire --tactile false

# Evaluate cross-morphology transfer (train on Allegro, test on Shadow + Inspire)
python train.py --task T01 --method IL-04 --hand allegro
python eval.py  --task T01 --method IL-04 --hand shadow,inspire --checkpoint path/to/ckpt
```

---

## Repository Structure

```
dexbench/
├── tasks/                  # Task definitions (reward, reset, termination)
│   ├── tier1/              # T01–T03
│   ├── tier2/              # T04–T06
│   ├── tier3/              # T07–T08
│   └── tier4/              # T09–T11
├── methods/                # Method implementations
│   ├── rl/                 # RL-01 ~ RL-06
│   ├── il/                 # IL-01 ~ IL-06
│   └── vla/                # VLA-01 ~ VLA-05
├── envs/                   # Simulation environment wrappers
│   ├── isaac_lab/
│   └── mujoco/
├── hands/                  # Hand URDF/MJCF assets and configs
├── tactile/                # Tactile sensor simulation and encoding
├── eval/                   # Evaluation scripts and metric logging
├── configs/                # Experiment configs (YAML)
├── scripts/                # Training and evaluation entry points
├── docs/                   # Documentation
├── README.md
└── README_zh.md
```

---

## Roadmap

- [x] Task suite definition (T01–T11)
- [x] Method interface specification
- [ ] Isaac Lab environment implementations (T01–T03, T09)
- [ ] MuJoCo environment implementations (T04, T06, T08)
- [ ] RL baseline implementations (RL-01, RL-02)
- [ ] Diffusion Policy baseline (IL-01)
- [ ] Tactile sensor simulation module
- [ ] Cross-morphology evaluation pipeline
- [ ] VLA integration (VLA-01, VLA-02)
- [ ] Full leaderboard and result tables
- [ ] Real-robot sim-to-real transfer guide

---

## Citation

If you use DexBench in your research, please cite:

```bibtex
@misc{dexbench2025,
  title   = {DexBench: A Simulation Benchmark for Dexterous Hand Manipulation},
  author  = {Your Name and Collaborators},
  year    = {2025},
  url     = {https://github.com/Barristen/dexbench}
}
```

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

## Acknowledgements

DexBench builds on or draws inspiration from the following open-source projects and research works: Isaac Lab, MuJoCo Playground, OpenVLA-OFT, Diffusion Policy, CrossDex, DexMimicGen, ForceVLA, Reactive Diffusion Policy, DexterityGen, and DexNDM. We thank all authors for making their code and data publicly available.
