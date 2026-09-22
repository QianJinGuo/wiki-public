---
title: "Recurrent Looped Transformer (RLT-1)"
created: 2026-09-22
updated: 2026-09-22
type: entity
tags: [architecture, transformer, efficiency]
sources: [raw/articles/yifanzhang-rlt1-recurrent-looped-transformer]
confidence: 0.7
---

# Recurrent Looped Transformer (RLT-1)

## 摘要

RLT-1 通过 gated cross-token recurrence 将上一 token 最终 decoder state 注入当前 token decoder input，计算深度随序列长度扩展；parity/permutation-tracking 任务多 seed 量化验证（与 Raschka 对 Astra looped 的 debunk 形成对照）^[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer.md]

## 核心要点

- v*c=49（value=7, confidence=7, stars=3），newsletter ingest 2026-09-22
- 详细分析见原文存档

相关：[[raw/articles/openai-astra-looped-transformer-debunk-raschka-2026]]

→ [[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer|原文存档]]
