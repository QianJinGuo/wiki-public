---
title: "知识 Provenance 方法对比"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, provenance, knowledge, source, trace, wiki]
sources: [concepts/source-first-knowledge-compilation, concepts/memory-source-provenance, entities/hermes-wiki-9-step-auto-growing-knowledge-network]
---

## 对照表

| 维度 | Hermes-Wiki source-first | Agent memory 三元组 | Git log | AI 摘要无 provenance |
|------|---|---|---|---|
| 机制 | caret 脚注绑定 raw/ 路径 | source_type + source_id + timestamp | git blame 文件级追溯 | ❌ 无来源标注 |
| 覆盖 | wiki 内容 | agent 长期记忆 | 代码 + 文档 | AI 直接产出 |
| 强制度 | lint 校验，0 errors 是 gate | schema 层强制 | git 自动 | 无 |
| 三月后可信度 | 高 | 高 | 中（只 track 改动） | 极低 |
| 失效检测 | lint 检测失链 | source 删除时标 unverified | git log 永存 | N/A |
| 适用阶段 | 知识库长期沉淀 | agent 实时记忆 | 代码协作 | ❌ 不推荐 |

## 判断

推荐组合：知识库用 Hermes-Wiki source-first（lint 强制） + agent 记忆用 Memory provenance（三元组） + 代码用 git。AI 摘要不带 source 是「未来不可信资产」——3 个月后回看，分不清是事实还是 LLM 编的。

## 对比方来源

- [[concepts/source-first-knowledge-compilation|source-first 编译]]
- [[concepts/memory-source-provenance|memory provenance]]
- data quality pipeline
- [[entities/hermes-wiki-9-step-auto-growing-knowledge-network|Hermes-Wiki 9 步法]]

## 进一步阅读

- [[concepts/source-first-knowledge-compilation]]
- [[concepts/memory-source-provenance]]
- [[entities/hermes-wiki-9-step-auto-growing-knowledge-network]]
