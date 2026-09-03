---

title: "深入浅出 Harness Engineering 之核心模式与理念"
created: 2026-06-10
updated: 2026-08-29
tags: [agent, architecture, claude, code, data, evaluation, harness-engineering, k8s, llm, memory, mlops, prompt, rag, search, security, tool-use, workflow]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/harness-engineering-core-patterns-claude-code
---

# 深入浅出 Harness Engineering 之核心模式与理念


## 相关实体

- [[entities/claude-code-large-codebase-agent-harness-13-patterns-tuutuiagi|面向大型代码库的 claude code 团队落地经验与扩展策略（agent harness）]]
- [[entities/claude-code-large-codebase-team-deployment-agent-harness|面向大型代码库的 claude code 团队落地经验与扩展策略（agent harness）]]
- [[entities/prosemirror-knowledge-base-mention-vivo|知识库问答 @文档：从 dom 方案到 prosemirror 落地]]
→ [[raw/articles/harness-engineering-core-patterns-claude-code|原文存档]] ^[raw/articles/harness-engineering-core-patterns-claude-code.md]

- [[moc/data-infrastructure|MOC]]
## 深度分析

深入浅出 Harness Engineering 之核心模式与理念 涉及agent领域的核心技术议题。 ^[raw/articles/harness-engineering-core-patterns-claude-code.md]
### 核心观点
1. # 深入浅出 Harness Engineering 之核心模式与理念 ^[raw/articles/harness-engineering-core-patterns-claude-code.md]
**作者：** 张碧泉 ^[raw/articles/harness-engineering-core-patterns-claude-code.md]
**发布日期：** 2026年4月29日 ^[raw/articles/harness-engineering-core-patterns-claude-code.md]
从 Claude Code、Claude Managed Agents、Hermes 三个系统出发，梳理 Harness Engineering 的核心模式：持久化指令、分层记忆、工作流编排、工具权限管理、Session/Harness/Sandbox 三件套解耦、凭证安全设计、多智能体协作模式、性能优化等。 ^[raw/articles/harness-engineering-core-patterns-claude-code.md]
2. ## 一、Claude Code 的核心模式 ^[raw/articles/harness-engineering-core-patterns-claude-code.md]
### 1.
3. 1 持久化指令文件 ^[raw/articles/harness-engineering-core-patterns-claude-code.md]
没有持久化指令文件时，每次对话都像从头开始，相同规则和错误反复出现。 ^[raw/articles/harness-engineering-core-patterns-claude-code.md]
4. 代价：文件需要随项目更新维护，否则可能误导智能体。 ^[raw/articles/harness-engineering-core-patterns-claude-code.md]
5. 2 作用域上下文组装 ^[raw/articles/harness-engineering-core-patterns-claude-code.md]
将指令按不同范围（组织、项目）拆分，让智能体动态加载最相关规则。 ^[raw/articles/harness-engineering-core-patterns-claude-code.md]

### 关联实体

- [[entities/你不知道的-agent原理架构与工程实践-v2]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/一文带你弄懂-ai-圈爆火的新概念harness-engineering]]
- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr]]
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]

