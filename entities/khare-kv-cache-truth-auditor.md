---
title: "A cache hit is not proof that you skipped the work"
created: 2026-09-22
updated: 2026-09-22
type: entity
tags: [kv-cache, inference, correctness, auditing]
sources: [raw/articles/khare-kv-cache-truth-auditor]
confidence: 0.7
---

# A cache hit is not proof that you skipped the work

## 摘要

KV-cache 正确性审计工具：cache hit 不等于 prefix 复用——确定性测试暴露 exact-dup/interior-mutation/namespace-isolation 三种 reuse 语义，cached 与 no-cache 输出 token-identical 验证^[raw/articles/khare-kv-cache-truth-auditor.md]

## 核心要点

- v*c=49（value=7, confidence=7, stars=4），newsletter ingest 2026-09-22
- 详细分析见原文存档

相关：[[raw/articles/mooncake-cluster-level-kv-cache-pooling-production-2026-09-11]]

→ [[raw/articles/khare-kv-cache-truth-auditor|原文存档]]
