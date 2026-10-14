---
title: "RoboFoundry：Self-Evolving System-as-Policy 具身框架"
created: 2026-10-14
updated: 2026-10-14
type: entity
tags: [embodied-ai, agent-harness, self-evolution, robotics, system-as-policy]
sources: [raw/articles/robofoundry-self-evolving-system-as-policy-2026]
confidence: 0.7
---

# RoboFoundry：Self-Evolving System-as-Policy 具身框架

NTU、北航、NUS、上交与云蝶科技联合提出，定位为首个把智能体系统本身形式化为「自进化的系统即策略」（Self-Evolving System-as-Policy）的具身框架：**基座模型冻结，进化发生在模型周围的上下文系统、技能系统与执行接口**，借助共享语义接口在不同机器人间复用 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md]。

## 叙事支点：从 Code-as-Policy 到 System-as-Policy

EmbodiedBench 上同一个 GPT-5.5：直接使用平均成功率 56.9%；不改一个参数、只让周围系统持续进化 → 72.7%。差距全部来自模型之外：哪些信息进入上下文、哪些经验被保留、决策如何落到物理动作。

与 [[entities/agent-self-evolution-flywheel-tencent-2026]]（单组件记忆/技能进化）的区别：RoboFoundry 把**整套系统**当作可进化对象，且诊断信号来自执行中暴露的问题（而非仅用于评测）。与浙大 [[entities/hugagentos-zju-self-evolution-agent-os-2026]] 同属「系统级自进化」家族，但本作聚焦具身 + 跨机器人语义接口复用。

## 关联
- → [[raw/articles/robofoundry-self-evolving-system-as-policy-2026|原文存档]]
