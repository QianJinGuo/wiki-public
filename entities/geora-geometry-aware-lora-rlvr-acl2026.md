---

title: "GeoRA: Geometry-Aware Low-Rank Adaptation for RLVR"
created: 2026-08-30
updated: 2026-09-05
type: entity
tags: [lora, rlvr, fine-tuning, reinforcement-learning, acl2026, low-rank]
sources: [raw/articles/geora-geometry-aware-lora-rlvr-acl2026]
confidence: 0.85
---

# GeoRA: Geometry-Aware Low-Rank Adaptation for RLVR

ACL 2026 杰出论文。美团履约技术团队提出了一种专为 RLVR（Reinforcement Learning from Verifiable Rewards）设计的低秩训练方法。 ^[raw/articles/geora-geometry-aware-lora-rlvr-acl2026.md]

## 核心问题

RLVR 场景下，标准 LoRA 的低秩约束与 RL 训练的几何特性不匹配，导致训练效率低下。

## GeoRA 方法

- **Geometry-Aware**：考虑了 RLVR 训练中参数空间的几何结构
- 专为 RLVR 设计的低秩适配方法，在保持参数效率的同时提升训练效果
- 在业务 Agentic RL 中有落地经验

## 意义

- ACL 2026 杰出论文（全球仅 18 篇）
- 为 RLVR + LoRA 的组合提供了理论基础和实践方案
- 美团履约技术团队在 Agentic RL 中的实际部署验证

## 相关论文

- 论文：[GeoRA: Geometry-Aware Low-Rank Adaptation for RLVR](https://aclanthology.org/2026.acl-long.1110/)

→ [[raw/articles/geora-geometry-aware-lora-rlvr-acl2026|原文存档]] ^[raw/articles/geora-geometry-aware-lora-rlvr-acl2026.md]