---
title: "Specification-Driven vs Vibe Coding：两种 AI 开发方法论对比"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, sdd, vibe-coding, methodology, ai-development]
sources: [concepts/specification-driven-agent-development, concepts/vibe-coding-paradigm, concepts/sdd-specification-driven-development-harness]
---

## 对照表

| 维度 | SDD (Specification-Driven) | Vibe Coding |
|------|---|---|
| 先做什么 | 写 spec（输入/输出/约束） | 直接写 prompt 跑 |
| 验证方式 | spec 派生测试用例，自动验证 | 人类目测接受/拒绝 |
| 成功率（复杂任务） | 80%+ | 30% |
| 前期投入 | 高（写 spec 2-10× 编码时间） | 低（prompt 即跑） |
| 后期维护 | 低（spec 是活文档 + test） | 高（没人知道为什么写成这样） |
| 适用场景 | 复杂项目、多人协作、长期维护 | 原型、脚本、一人玩具 |
| 典型代表 | Anthropic Claude Code SDD 实践 | Karpathy 2025 vibe coding 倡议 |

## 判断

并存而非替代——最佳组合是「vibe 探索 → 确认需求 → SDD 正式实现」。SDD 一上来就用会扼杀创意；vibe 一直用会在严肃项目失败。判断信号：当你发现「这个原型需要持续维护」的瞬间，就该把 vibe 阶段产出的 prompt 翻译成 SDD spec。

## 对比方来源

- [[concepts/specification-driven-agent-development|SDD agent dev concept]]
- [[concepts/vibe-coding-paradigm|vibe coding 范式]]
- [[concepts/verifier-driven-development|verifier-driven development]]
- [[concepts/agentic-engineering-paradigm|agentic engineering]]

## 进一步阅读

- [[concepts/specification-driven-agent-development]]
- [[concepts/vibe-coding-paradigm]]
- [[concepts/sdd-specification-driven-development-harness]]
