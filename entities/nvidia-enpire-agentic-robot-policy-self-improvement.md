---

title: "Agentic Robot Policy Self-Improvement in the Real World"
description: "NVIDIA/CMU/Berkeley 的真实世界机器人策略自改进系统 ENPIRE"
source: "[[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement]]"
tags:
  - robot-learning
  - self-improvement
  - nvidia
  - agentic
  - real-world-robotics
  - policy-learning
created: 2026-06-22
updated: 2026-08-01
type: entity
review_value: 9
review_confidence: 9
review_recommendation: strong
review_stars: 5
sources:
  - raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement
---

# Agentic Robot Policy Self-Improvement in the Real World

> 原文存档：[[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement|原文存档]]

## 核心内容


Published Time: Wed, 17 Jun 2026 23:48:33 GMT^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]


Markdown Content:
## ENPIRE: Agentic Robot Policy Self-Improvement in the Real World

, [Jia Xie](https://jia-xie.com/)2†, [Tonghe Zhang](https://tonghe-zhang.github.io/)2†, [Haotian Lin](https://darthutopian.github.io/)2†, [Letian "Max" Fu](https://max-fu.github.io/)3, [Haoru Xue](https://haoruxue.github.io/)3, Jalen Lu 2, ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

[Yi Yang](https://yiyang-23.github.io/)2, [Cunxi Dai](https://cunxid.github.io/)2, [Zi Wang](https://ziwang1105.github.io/)1, [Jimmy Wu](https://jimmyyhwu.github.io/)1, [Guanzhi Wang](https://guanzhi.me/)1, [S. Shankar Sastry](https://www2.eecs.berkeley.edu/Faculty/Homepages/sastry.html)3, [Ken Goldberg](https://goldberg.berkeley.edu/)3, ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

[Linxi "Jim" Fan](https://jimfan.me/)1‡, [Yuke Zhu](https://yukezhu.me/)1‡, [Guanya Shi](https://www.gshi.me/)2‡ ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

1 NVIDIA 2 CMU 3 UC Berkeley†Equal contribution‡Equal advising ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

[Paper](https://drive.google.com/drive/folders/1J8w1yQux9ODYqTNZ2ynOIFjerBIQtw1V?usp=sharing) ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

![Image 1: NVIDIA](http://research.nvidia.com/labs/gear/enpire/images/logos/nvidia-logo.png)![Image 2: Carnegie Mellon University](http://research.nvidia.com/labs/gear/enpire/images/logos/cmu-logo.png)![Image 3: UC Berkeley](http://research.nvidia.com/labs/gear/enpire/images/logos/uc-berkeley-logo.png) ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

## Abstract

Achieving dexterous robotic manipulation in the real world relies heavily on human supervision and algorithmic engineering, which is a central bottleneck in the pursuit of general physical intelligence. Although emerging coding agents can generate code to automate algorithm search, their successes remain largely confined to digital environments. We conjecture that the missing abstraction to automate robotics research is a repeatable feedback loop for real-world policy improvement: reset the scene, execute a policy, verify the outcome, and refine the next iteration. ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

To bridge this gap, we introduce ENPIRE, a harness framework for coding agents that instantiates this physical feedback routine with four core modules: an Environment module (EN) for automatic reset and verification, a Policy Improvement module (PI) that launches policy refinement, a Rollout module (R) to evaluate policies with single or multiple physical robots operating in parallel, and an Evolution module (E) in which coding agents analyze logs, consult literature, improve training infrastructure and algorithm code to address failure modes. ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

This closed-loop system transforms real-world robot learning into a controllable optimization procedure that agents can manage, thus minimizing human effort while allowing fair ablations across training recipes and agent variants. Powered by ENPIRE, frontier coding agents can autonomously develop a policy to achieve a 99% success rate on challenging, dexterous manipulation tasks in the real world, such as PushT, organizing pins into a pin box, and using a cutter to cut a zip tie. ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

Coding agents can improve policies with various PI regimes, such as heuristic learning, tool calling, behavior cloning, offline or online RL. Moreover, ENPIRE can be significantly accelerated on a robot fleet, and we propose two metrics, namely, Mean Robot Utilization (MRU) and Mean Token Utilization (MTU) to measure the efficiency of multiagent physical autoresearch. We also include simulation results in RoboCasa. Our findings suggest a practical and scalable path toward autonomously advancing robotics in the real world. ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

## Learned Manipulation Policy

Policies trained with ENPIRE reach a 99% pass@8 success rate across the showcased manipulation tasks. ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

ENPIRE runs fully autonomously on real robots. Working only through the automated reset and verification interface, a team of coding agents proposes algorithmic hypotheses (heuristic learning, behavior cloning, offline and online RL), tests them against the real-world success rate, and keeps the changes that move it. The idea tree below traces that search as a hypothesis git-tree — one branch per agent, one node per idea tried — plotted on the same wall-clock-time axis as the success-rate curve, so you can see the ideas that moved the curve upward. ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

**Figure 1:**Each coding agent explores its own branch of ideas, one lane per branch. Every dot is an idea it tried; a green ring marks an idea that raised the team’s average success rate, and green curves trace cross-agent inspiration. The lower panel tracks the team’s average success rate climbing over research wall-clock time. ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

## ENPIRE System

Construct Environment^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]


Policy Improvement

Action

Obs

Reward

env.py

1

class InsertionEnv:

2

def reset(self):

3

# TODO: auto task reset

4

pick_and_place(obj, target)^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]


5

go_home()

6

...

7

8

def get_reward(self, obs, act):^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]


9

# TODO: scalar reward

10

mask = sam3(obs['left'])^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]


11

pos = boundlsdf(obs, mask)^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]


12

...

13

14

def get_observation(self):^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]


15

...

16

17

def step(self, act):^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]


18

...

Human User

Coding Agent

Tool APIs

![Image 4: Perception — SAM 3](http://research.nvidia.com/labs/gear/enpire/sam.min.jpg) ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

Perception

![Image 5: Planning — cuRobo](http://research.nvidia.com/labs/gear/enpire/curobo.min.jpg) ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

Planning

![Image 6: Control — YAM arm](http://research.nvidia.com/labs/gear/enpire/control.min.jpg) ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

Control

![Image 7: a real robot farm](http://research.nvidia.com/labs/gear/enpire/robot_farm.min.jpg)ENPIRE Environment ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

01 Literature review^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]


PLD RL-Token CaP-X

02 Propose algorithm variant^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]


Heuristics Off2On RL Code-as-policy BC^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]


03 Optimize Infra

Data Sampler Param Sweep^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]


04 Summarize experiment result^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]


Hillclimb Timeline

![Image 8: GPU insertion](http://research.nvidia.com/labs/gear/enpire/gpu_insertion.min.jpg) ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

GPU insertion

![Image 9: Pin insertion](http://research.nvidia.com/labs/gear/enpire/pin_insertion_2.min.jpg) ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

Pin insertion

![Image 10: Push-T](http://research.nvidia.com/labs/gear/enpire/push_t.min.jpg) ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

Push-T

![Image 11: Zip tie cutting](http://research.nvidia.com/labs/gear/enpire/zip_tie.min.jpg) ^[raw/articles/nvidia-enpire-agentic-robot-policy-self-improvement.md]

Zip tie cutting

Real-world t

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

