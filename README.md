# A COLREGs-compliant multi-ship collision avoidance approach based on Deep Reinforcement Learning

**NUS IE4100R Final Year Project** · March 2023  
**Author:** He Zhenyu (A0205505R) · **Supervisor:** Dr. Li Haobin

This repository contains **partial thesis / experiment code** for an undergrad FYP on autonomous multi-ship collision avoidance that stays aligned with **COLREGs** (Convention on the International Regulations for Preventing Collisions at Sea), trained with **deep reinforcement learning**.

## Problem

Busy seaways create high-risk encounters (head-on, crossing, overtaking). COLREGs rules are intentionally broad and often interpreted subjectively, while many existing autonomous methods either:

- ignore COLREGs, or
- optimise a **single** ship’s safety in 1-to-many settings (which can push risk onto other vessels)

Real traffic is multi-agent: ships interact together, not as one ego vessel dodging static others.

## Approach

Build a custom multi-ship Gym environment and train DRL agents so vessels reach destinations while:

- avoiding collisions
- respecting COLREGs steering/sailing behaviour where possible
- managing encounter risk (e.g. ship domain, CPA / TCPA / DCPA-style signals in the reward design)

### Environment highlights

| Item | Detail |
|------|--------|
| Ships | Configurable; **max 8**; FYP training/testing focused on **4-ship** open-sea scenarios |
| Observation | State features per ship (position, speed, size, destination, heading/bearing, arrival status) — `n × 9` |
| Actions | Accelerate, decelerate, steer clockwise / anti-clockwise, or do nothing |
| Risk signals | Ship domain, TCPA, DCPA used in reward shaping |
| Training budget | Up to **30M** environment timesteps (project limit) |

### Algorithms explored

- **DQN** (off-policy)
- **PPO** (on-policy)
- **A2C** (actor–critic)

The notebook uses **Stable-Baselines3** (`PPO`, `DQN`, `A2C`) with TensorBoard logging.

Reward design was a major focus: collision / time-step / risk / COLREGs-violation penalties, plus destination progress and arrival rewards — tuned to avoid local optima and undesired behaviours (e.g. circling, over-avoidance, intentional early collision under a harsh time penalty).

An **action-masking** variant (block illegal COLREGs actions instead of only penalising them) was also studied; it reduced COLREGs violations in tests but increased collisions in the reported evaluation setup.

## What’s in this repo

| File | Role |
|------|------|
| `HEZHENYU_fypcodeused.ipynb` | Main FYP code used: custom env, helpers (distance / TCPA–DCPA), training loops |
| `README.md` | Project overview |

> This is **partial** project code (not a full paper/code release). Presentation materials from the FYP viva are separate from this repo.

## Quick start

1. Use a Python environment with the notebook dependencies (e.g. `gym`, `numpy`, `tensorflow` / `torch`, `stable-baselines3`, `matplotlib`, `pygame` for render modes).
2. Open `HEZHENYU_fypcodeused.ipynb`.
3. Adjust env settings near the top (e.g. `NUMBER_OF_SHIPS`, map size, reward weights) before training.

```python
NUMBER_OF_SHIPS = 4   # max 8
MAP = 200
```

## Limitations (as noted in the FYP)

- Training/evaluation centred on **4-ship** open-sea scenarios
- Actions assumed to affect ship motion immediately (simplified dynamics)
- Compute capped at **30M** timesteps
- More realistic ship physics would be a natural next step

## Citation / context

Module **IE4100R** · B.Eng dissertation presentation · **9 Mar 2023** · National University of Singapore
