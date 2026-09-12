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

Build a custom multi-ship Gym environment (`ssship` in the notebook) and train DRL agents so vessels reach destinations while:

- avoiding collisions
- respecting COLREGs steering/sailing behaviour where possible
- managing encounter risk (ship domain / TCPA / DCPA-style signals in the reward)

### Environment overview

| Item | Detail |
|------|--------|
| Ships | Configurable via `NUMBER_OF_SHIPS` (code supports **up to 8**; FYP runs focused on **4**) |
| Map | Square open-sea grid; default `MAP = 200` scaled by `SCALAR = 2` for rendering |
| Episode length | `TIMESTEP = 2000` steps (episode ends earlier on all-arrive / collision) |
| Framework | OpenAI Gym-style API + pygame render (`human` / `rgb_array`) |
| Training stack | Stable-Baselines3 (`PPO`, `DQN`, `A2C`) + TensorBoard |

### Observation space

Each step returns a state array of shape **`(n_ships, 9)`**.

In the notebook env, ship `i` is stored as:

| Index | Feature | Notes |
|------:|---------|--------|
| 0 | `x` | Position on the map |
| 1 | `y` | Position on the map |
| 2 | `speed` | Initialised roughly in `[0.5, 1.5]` |
| 3 | `bearing` | Heading angle in degrees `[0, 360)` |
| 4 | `Lx` | Destination x |
| 5 | `Ly` | Destination y |
| 6 | `arrived` | `0` / `1` flag |
| 7 | `length` | Ship length (spawned small ints) |
| 8 | `width` | Ship width |

So the joint observation is an **`n × 9`** matrix of per-ship kinematic / goal / size features (not raw camera frames). Destinations are sampled so they are not too close to the spawn point; ships are also spawned with a minimum separation.

Declared Gym space (as in code):

```python
observation_space = spaces.Box(0, MAP / SCALAR, shape=(n_ships, 9), dtype=int)
```

### Action space

One discrete control is chosen **per ship** each step (joint action length = `n_ships`).

| Action id | Effect in `step()` |
|----------:|--------------------|
| `0` | Steer **+15°** (bearing increases) |
| `1` | Steer **−15°** (bearing decreases) |
| `2` | **Accelerate** (`speed += 0.1`) |
| `3` | **Decelerate** (`speed -= 0.1`) |
| `4` | **Do nothing** |

Motion update (simplified kinematics): position advances from current speed and bearing; map edges clamp / slide rather than wrap freely. Arrived ships (`arrived == 1`) no longer apply actions.

Conceptually this matches the FYP presentation’s five controls: steer clockwise / anti-clockwise, accelerate, decelerate, idle.

### Rewards & risk (short)

Shaped from several terms (weights are notebook hyperparameters such as `Collision`, `Arrival`, `TSP`, `violation`):

- per-step time penalty
- arrival reward when within destination radius
- collision penalty + episode terminate when ships get too close
- continuous risk penalty from TCPA/DCPA when ships are near
- COLREGs-related penalties for head-on / crossing / overtaking give-way situations

Reward design was a major focus: avoid local optima and bad behaviours (circling to farm progress reward, over-avoidance, intentional early collision under a harsh time penalty).

An **action-masking** variant (block illegal COLREGs actions instead of only penalising them) was also studied; it cut COLREGs violations in tests but raised collisions in the reported evaluation setup.

### Algorithms explored

- **DQN** (off-policy)
- **PPO** (on-policy)
- **A2C** (actor–critic)

Project training budget discussed in the FYP: up to **~30M** environment timesteps.

## What’s in this repo

| File | Role |
|------|------|
| `HEZHENYU_fypcodeused.ipynb` | Main FYP code used: custom env, TCPA/DCPA helpers, COLREGs checks, training loops |
| `README.md` | Project overview |

> This is **partial** project code (not a full paper/code release). Presentation materials from the FYP viva are separate from this repo.

## Quick start

1. Use a Python environment with the notebook dependencies (e.g. `gym`, `numpy`, `tensorflow` / `torch`, `stable-baselines3`, `matplotlib`, `pygame` for render modes).
2. Open `HEZHENYU_fypcodeused.ipynb`.
3. Adjust env settings near the top before training:

```python
TIMESTEP = 2000
NUMBER_OF_SHIPS = 4   # max 8
SCALAR = 2
MAP = 200
```

## Limitations (as noted in the FYP)

- Training/evaluation centred on **4-ship** open-sea scenarios
- Actions assumed to affect ship motion immediately (simplified dynamics)
- Compute capped at **30M** timesteps
- More realistic ship physics would be a natural next step

## Citation / context

Module **IE4100R** · B.Eng dissertation presentation · **9 Mar 2023** · National University of Singapore
