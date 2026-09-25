---
title: "Intelligence Density：Trajectory 提出「单位智能成本」训练目标"
created: 2026-09-26
updated: 2026-09-26
type: entity
tags: [post-training, rl, efficiency, evaluation, density-aware-training, token-efficiency, nemotron]
sources: [raw/articles/intelligence-density-trajectory-density-aware-training-2026]
confidence: 0.75
provenance_state: extracted
---

# Intelligence Density：Trajectory 提出「单位智能成本」训练目标

Trajectory 提出 intelligence density（cost-per-task 而非 cost-per-token）作为评估与训练目标：更便宜的 token ≠ 更便宜的任务。density-aware training 在 Nemotron 3.5 Nano 30B 上保持 8.3% pass率同时把 mean output 从 90k tokens 压到 37k（Legal Agent Benchmark）；Tau3 实验显示 reward 设计改变模型学到的行为（partial-credit vs strict accuracy）。 ^[raw/articles/intelligence-density-trajectory-density-aware-training-2026.md]

## 相关链接

- [[concepts/inference-optimization|推理优化（成本侧）]]
- [[entities/agent-eval-counterintuitive-insights-langfuse|Agent 评估反直觉洞察]]

→ [[raw/articles/intelligence-density-trajectory-density-aware-training-2026|原文存档]]
