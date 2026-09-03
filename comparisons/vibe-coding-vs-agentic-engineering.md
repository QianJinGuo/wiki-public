---
title: "vibe coding vs agentic engineering：Karpathy 范式迁移"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, vibe-coding, agentic-engineering, karpathy, paradigm]
sources: [entities/karpathy-vibe-coding-agentic-engineering, entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering, entities/karpathy-vibe-coding-to-agentic-engineering]
---

## 对照表

| 维度 | vibe coding (2025) | agentic engineering (2026) |
|------|---|---|
| 核心口号 | 让 AI 帮你写出来 | 让 AI 在可验证系统里把东西交付出去 |
| 控制粒度 | 不关心细节，只看 vibe | harness/context/verifier 全链路控制 |
| 验证方式 | 人类目测接受/拒绝 | 自动 verifier + 人工 review gate |
| 适用边界 | 原型/脚本/玩具 | 生产/安全敏感/长期维护 |
| 模型假设 | 模型越来越强就好用 | 模型强≠系统可靠，harness 决定上限 |
| 工程债务 | 低（原型即抛弃） | 高（需持续维护 harness/verifier） |
| 交付物 | 可演示的功能片段 | 可问责的、可监控的、可回滚的系统 |

## 判断

vibe coding 不是错——它解决了「快速做出来」。agentic engineering 是它的工程化升级，解决「可靠交付」。两者不是非此即彼：用 vibe 探索可能性，确认方向后用 agentic 落地。Karpathy 本人最近的表态：vibe coding「已死」是指它作为最终范式已死，作为入口仍然有用。

## 对比方来源

- [[concepts/vibe-coding-paradigm|vibe coding 范式]]
- [[concepts/agentic-engineering-paradigm|agentic engineering 范式]]
- [[concepts/verifier-driven-development|verifier-driven development]]
- [[concepts/software-3-0-stack|Software 3.0 stack]]

## 进一步阅读

- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/karpathy-vibe-coding-to-agentic-engineering]]
