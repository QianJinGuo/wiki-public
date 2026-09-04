---

title: "AI Agent 工程师学习路线：面向资深后端/大数据工程师的能力地图"
created: 2026-06-10
updated: 2026-09-05
tags: [agent, architecture, code, data, database, evaluation, llm, memory, mlops, prompt, rag, search, security, tool-use, workflow]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/ai-agent-engineer-learning-roadmap-backend-2026
---

# AI Agent 工程师学习路线：面向资深后端/大数据工程师的能力地图

→ [[raw/articles/ai-agent-engineer-learning-roadmap-backend-2026|原文存档]] ^[raw/articles/ai-agent-engineer-learning-roadmap-backend-2026.md]

## 深度分析

AI Agent 工程师学习路线：面向资深后端/大数据工程师的能力地图 ^[raw/articles/ai-agent-engineer-learning-roadmap-backend-2026.md]
### 核心观点
1. # AI Agent 工程师学习路线：面向资深后端/大数据工程师的能力地图 ^[raw/articles/ai-agent-engineer-learning-roadmap-backend-2026.md]
## 核心判断
**AI Agent 不是 Prompt 工程的延长线，而是一套新的应用工程体系。 ^[raw/articles/ai-agent-engineer-learning-roadmap-backend-2026.md]
2. ** ^[raw/articles/ai-agent-engineer-learning-roadmap-backend-2026.md]
对后端/大数据工程师来说，这是优势区，不是劣势区。 ^[raw/articles/ai-agent-engineer-learning-roadmap-backend-2026.md]
3. 模型能力层 ^[raw/articles/ai-agent-engineer-learning-roadmap-backend-2026.md]
- 结构化输出、Tool Calling、推理边界、长上下文
- 小模型做分类/抽取/路由；中模型做常规工具选择；大模型做复杂推理
- 生产级优化：任务分层、模型路由、缓存、上下文治理
### 2.
4. 上下文与知识层（RAG升级） ^[raw/articles/ai-agent-engineer-learning-roadmap-backend-2026.md]
- RAG = Agent的外部知识供给机制，不只是知识库问答
- 可服务：业务文档、历史案例、代码库片段、内部SOP、工单记录、日志片段
- 关键问题：query rewrite、multi-query retrieval、hybrid retrieval、rerank、长上下文配合
### 3.
5. 记忆层（架构问题，不是聊天记录回填） ^[raw/articles/ai-agent-engineer-learning-roadmap-backend-2026.md]
- **Working Memory**：当前任务运行态——步骤、中间推理结果、工具返回值、临时变量
- **Session Memory**：单会话周期内——用户目标、偏好、约束条件、任务进度
- **Long-Term Memory**：跨会话——用户画像、历史成功/失败案例、可复用策略、偏好
### 4.

### 关联实体

- [[entities/你不知道的-agent原理架构与工程实践-v2]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏-v2]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/一文带你弄懂-ai-圈爆火的新概念harness-engineering]]

## 相关实体

- [[moc/mlops-training-inference|MOC]]
