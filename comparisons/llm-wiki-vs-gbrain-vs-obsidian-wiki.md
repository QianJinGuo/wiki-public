---
title: "LLM Wiki vs GBrain vs Obsidian-Wiki：自组织知识库三实现对比"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, llm-wiki, gbrain, obsidian-wiki, knowledge-base, self-organization]
sources: [entities/llm-wiki-obsidian-wiki-gbrain-self-organization-self-evolution, entities/llm-wiki-architecture, entities/karpathy-llm-wiki-v2-2026]
---

## 对照表

| 维度 | LLM Wiki (Karpathy) | GBrain (System F) | Obsidian-Wiki (社区) |
|------|---|---|---|
| 核心范式 | wiki 结构 + LLM 编译器 | 知识图谱 + 嵌套标签 | Obsidian 双链 + 插件 |
| 存储单元 | .md + wikilink | 图谱节点 + 标签层级 | .md + Obsidian 元数据 |
| AI 集成度 | 高（LLM 自动拆解/编译/关联） | 中（AI 辅助查询） | 低-中（插件集成 LLM） |
| 网络机制 | 强制 wikilink + 5 点验收 | 标签层级自动聚类 | backlink + 图谱视图 |
| 可操作性 | Hermes-Wiki 9 步法落地 | 概念原型 | 开箱即用 + 插件市场 |
| 学习曲线 | 中-高 | 高（嵌套标签哲学复杂） | 低 |
| 适合谁 | AI 研究者 / 知识工作者 | 极客 / 知识工程师 | 普通用户 / 写作者 |

## 判断

LLM Wiki 范式（Karpathy 提出，Hermes-Wiki 落地）是当前最有工程可行性的：纯文本 + wikilink + Obsidian 编辑 + LLM 编译，本地可控、AI 集成深、上手适中。GBrain 太理想化、Obsidian-Wiki 缺统一范式。

## 对比方来源

- [[concepts/llm-wiki-paradigm|LLM Wiki paradigm]]
- [[concepts/knowledge-network-self-growth|知识网络自生长]]
- [[entities/karpathy-llm-wiki-v2-2026|Karpathy LLM Wiki v2]]
- [[entities/hermes-wiki-9-step-auto-growing-knowledge-network|Hermes-Wiki 9 步法]]

## 进一步阅读

- [[entities/llm-wiki-obsidian-wiki-gbrain-self-organization-self-evolution]]
- [[entities/llm-wiki-architecture]]
- [[entities/karpathy-llm-wiki-v2-2026]]
