---
title: 第 2 层全库索引：交互实践
created: 2026-06-24
updated: 2026-06-24
type: moc
tags: [learning-path, layer-2, interaction, prompt, rag]
layer: 2
---

# 第 2 层：交互实践 — 全库索引

> 返回 [学习路径总入口](learning-path.md)

---

## 本层导读

第 1 层让你理解「LLM 怎么工作」。第 2 层给你「怎么用好它」：Prompt 工程、Prompt 模式、上下文工程、RAG。这层是从「会用 ChatGPT」到「能用 LLM 造工具」的过渡。

---

## 学习路径

```
chap-9 Prompt 基础（25min）→ chap-10 Prompt 模式（50min）→ chap-11 上下文工程（50min）→ chap-12 RAG（75min）→ 🚪 关卡
```

---

## 本层 concepts

### Prompt 工程
- [[concepts/prompt-engineering-fundamentals|Prompt Engineering Fundamentals]]
- Prompt 工程模式
- [[concepts/claude-code-best-practices-prompt-engineering|Claude Code 最佳实践]]

### 上下文工程
- [[concepts/context-engineering|上下文工程]]
- [[concepts/context-management-agent-systems|Agent 上下文管理]]
- [[concepts/harness-context-window-management|Harness 上下文窗口管理]]

### RAG 与检索
- [[concepts/retrieval-augmented-generation-rag|RAG]]
- RAG 进阶
- 向量搜索与嵌入
- 知识图谱 RAG
- Agentic RAG 模式
- 搜索检索

### 长上下文
- 长上下文技术

---

## 本层 entities（精选）

### Prompt 实践
- [[entities/anthropic-prompt-caching-claude-code|Anthropic Prompt Caching 九原则]]
- [[entities/claude-code-prompt-source-analysis|Claude Code Prompt 源码]]
- [[entities/claude-code-prompt-context-harness|Claude Code 上下文 Harness]]

### 上下文工程
- [[entities/claude-code-context-engineering-anthropic-thariq|Anthropic 上下文工程]]
- [[entities/codex-context-engineering-lastwhisper-thinking-in-context|Codex 上下文解读]]
- [[entities/agent-harness-architecture-design-production-guide|Agent Harness 七层]]

### RAG 实践
- [[entities/rag-chunk-embedding-rerank-pipeline|RAG Pipeline]]
- [[entities/gemini-embedding-2-multimodal-unified-vector-hyman|Gemini Embedding 2]]
- [[entities/向量库是rag的前菜知识图谱是答案本体论是灵魂|向量库 vs 知识图谱]]

### 前沿
- [[entities/agent-memory-engineering-tax-aws-china-2026|Agent 记忆工程税]]
- [[entities/anthropic-12-mcp-production-patterns|12 个 MCP 生产模式]]

---

## 本层 raw

- [[entities/ai-coding-入门指南-如何更好地让ai真正帮你干活-v2|AI Coding 入门指南]]
- [[raw/articles/向量库是rag的前菜知识图谱是答案本体论是灵魂|向量库是 RAG 的前菜]]
- [[raw/articles/claude-code-prompt-source-analysis-fanone|Claude Code Prompt 源码分析]]
- [[raw/articles/claude-code-prompt-context-harness|Claude Code 上下文 Harness]]

---

## 🚪 关卡

1. **场景题**：用 Prompt 四组件设计「会议纪要转行动项」的 Prompt。
2. **费曼题**：向 12 岁孩子解释「Few-shot 和 Zero-shot 的区别」。
3. **关联题**：Prompt Engineering 和 Context Engineering 的本质区别？
4. **场景题**：RAG 检索准确率低，从嵌入、chunk、rerank 三方面优化。
5. **关联题**：回顾第 5 章上下文窗口。RAG 怎么「绕过」窗口限制？又引入什么新问题？

---

## 学完这层你应该能

- [ ] 写出结构清晰的 Prompt（四组件齐全）
- [ ] 在合适场景选择 Zero-shot / Few-shot / CoT
- [ ] 解释 Prompt Engineering 到 Context Engineering 的演进
- [ ] 画出 RAG 基本流程（嵌入→检索→重排→生成）
- [ ] 说出向量 RAG 和知识图谱 RAG 的区别

---

**下一层**：[第 3 层：Agent 工程](layer-3-agent-engineering.md)
