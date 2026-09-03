---

title: "告别“氛围编程”：基于 Harness 治理和 SDD 的团队级 AI 研发范式演进与实践"
created: 2026-05-10
updated: 2026-05-10
type: entity
tags: [ai-agent, engineering, llm, wechat]
review_value: 6
sources: []
review_confidence: 7
---

> -> [[raw/articles/告别氛围编程基于-harness-治理和-sdd-的团队级-ai-研发范式演进与实践.md|原文存档]]
从微信文章 [[raw/articles/告别氛围编程基于-harness-治理和-sdd-的团队级-ai-研发范式演进与实践.md|告别“氛围编程”：基于 Harness 治理和 SDD 的团队级 AI 研发范式演进与实践]] 提取。 ^[raw/articles/告别氛围编程基于-harness-治理和-sdd-的团队级-ai-研发范式演进与实践.md]

## 核心内容
source_url: https://mp.weixin.qq.com/s/-_IBJFuXpvoqMJxL9oaEJQ ^[raw/articles/告别氛围编程基于-harness-治理和-sdd-的团队级-ai-研发范式演进与实践.md]

### 主要章节
- ####  原因2：存量应用进行 Vibe Coding 风险非常高
- ####  原因3：大型项目、复杂需求超出了单次 AI 对话的能力边界
- ####  SDD  （规范驱动开发）
- ####  Harness Engineering  （驾驭工程）
- ####  步骤1：设计知识库
- ####  步骤2：处理需求 PRD
- ####  步骤3：专家团执行任务
- ####  步骤4：任务部署

## 深度分析
「氛围编程」（Vibe Coding）的本质是用自然语言模糊了需求和实现之间的边界，这在探索性、原型化阶段有巨大价值，但当系统进入存量维护和复杂需求阶段时，这种模糊性就成了风险本身。Harness Engineering 和 SDD 的提出，本质上是在说：AI 辅助开发需要一套结构化的「确定性承重层」（Deterministic Load-Bearing Layer），而不能把所有决策都推给模型的随机生成能力。 ^[raw/articles/告别氛围编程基于-harness-治理和-sdd-的团队级-ai-研发范式演进与实践.md]
文章中「存量应用进行 Vibe Coding 风险非常高」这一判断揭示了一个被忽视的工程现实：对于已有大量业务逻辑、边界条件和异常处理的历史系统，AI 的介入必须非常谨慎。一个 5 万行的遗留系统，其「隐性规则」数量远超显式代码，任何未经 Harness 过滤的 AI 生成代码都可能与这些隐性规则产生冲突，结果是系统性的、难以追踪的回归 Bug。 ^[raw/articles/告别氛围编程基于-harness-治理和-sdd-的团队级-ai-研发范式演进与实践.md]
SDD（规范驱动开发）和 Harness Engineering 的关系值得深究：SDD 提供的是需求到规范的转化层（PRD → 设计知识库），而 Harness Engineering 提供的是规范到代码的治理层（设计知识库 → AI 执行）。这两者合在一起，构成了一条从业务需求到 AI 生成结果的完整链路。 ^[raw/articles/告别氛围编程基于-harness-治理和-sdd-的团队级-ai-研发范式演进与实践.md]

## 实践启示
**团队引入 AI 辅助开发时**：第一步不是让所有人开始用 AI 写代码，而是建立设计知识库——把团队的设计决策、业务规则、边界条件用 AI 可消费的格式（而非自然语言文档）结构化存储。没有这个基础层，AI 生成的代码质量完全依赖模型能力，无法进行有针对性的治理。 ^[raw/articles/告别氛围编程基于-harness-治理和-sdd-的团队级-ai-研发范式演进与实践.md]
**存量项目改造**：采用「特权最小介入」原则——只让 AI 处理明确无误的任务（如测试生成、文档编写、重构辅助），关键业务逻辑和跨模块决策仍由人类把关。在 Harness 中设置「AI 禁止修改」区域，并配置强制人工 review 的门禁。 ^[raw/articles/告别氛围编程基于-harness-治理和-sdd-的团队级-ai-研发范式演进与实践.md]
**大型复杂项目**：将需求进行 SDD 分解后，再通过专家团（Human Expert Panel）逐个确认每个子任务的可行性和安全性。AI 的角色是「加速执行」，而非「替代判断」。任务部署前必须有 Harness 的自动化检查清单覆盖。 ^[raw/articles/告别氛围编程基于-harness-治理和-sdd-的团队级-ai-研发范式演进与实践.md]

- [[entities/harness-sdd-duiwu-ai-fullstack-dewux|基于 Harness + SDD + 多仓管理模式的 AI 全栈开发实践]]
- [[entities/harness不是目的知识才是护城河-一个ai工程交付团队的知识沉淀实践|Harness不是目的，知识才是护城河 —— 一个AI工程交付团队的知识沉淀实践]]
- [[entities/martin-fowler-ai-rd-harness-nondeterminism|Martin Fowler AI 研发 Harness：非确定性承重层]]
- [[entities/tencent-ai-team-knowledge-harness|腾讯 AI Team 知识沉淀体系（Harness Engineering 实践）]]

## 相关实体
- [[entities/agent-开发范式演进从环境工程出发简化多源实时上下文|Agent 开发范式演进：从环境工程出发，“简化”多源实时上下文]]
- [[entities/skillos-learning-skill-curation-for-self-evolving-agents|SkillOS: Learning Skill Curation for Self-Evolving Agents]]
- [[entities/深度拆解-hermes-agent-记忆系统它修正了-openclaw-的哪层误区|深度拆解 Hermes Agent 记忆系统：它修正了 OpenClaw 的哪层误区？]]
- [[entities/harness-engineering实践做了一个平台让ai一晚上自动评测和优化你的系统|Harness Engineering实践做了一个平台让AI一晚上自动评测和优化你的系统]]
- [[entities/cursor-复盘-harness模型决定能力上限harness-决定生产下限|Cursor 复盘 Harness：模型决定能力上限，Harness 决定生产下限]]
- [[entities/在-rds-postgresql-中实现-rabitq-量化|在 RDS PostgreSQL 中实现 RaBitQ 量化]]
- [[entities/codeindex-让大模型更好地理解你的代码|Codeindex · 让大模型更好地理解你的代码]]
- [[entities/使用-agent-skills-做知识库检索能比传统-rag-效果更好吗|使用 Agent Skills 做知识库检索，能比传统 RAG 效果更好吗？]]
- [[entities/rag深度解析分块向量化召回重排才是蒸馏同事skill的关键|RAG深度解析：分块、向量化、召回、重排，才是"蒸馏同事skill"的关键]]
- [[entities/claude-code-之父最新访谈编程已经结束harness-将消失claude-code-将只有-100-行代码loop-才是未来|Claude Code 之父最新访谈：编程已经结束、harness 将消失、Claude Code 将只有 100 行代码、loop 才是未来]]
- [[entities/harness-engineering耗时一周我是如何将应用的ai-coding率提升至90的|Harness Engineering：耗时一周，我是如何将应用的AI Coding率提升至90%的]]

- [[entities/anthropic-联创2028-年实现-ai-自我构建的概率超过-60|Anthropic 联创：2028 年实现 AI 自我构建的概率超过 60%]]
- [[entities/agent架构关键变化harness正在成为新后端|Agent架构关键变化：Harness正在成为新后端]]
- [[entities/我把-karpathy-的-autoresearch-搬到了软件开发领域效果炸了|我把 Karpathy 的 AutoResearch 搬到了软件开发领域，效果炸了]]
- [[entities/吴恩达ai-将最先杀死前端|吴恩达：AI 将最先杀死前端]]
- [[entities/精选-10-个开发者常用的-ai-智能体技能agent-skills|精选 10 个开发者常用的 AI 智能体技能（Agent Skills）]]
- [[entities/国产顶尖模型-benchmark-评分那么高可实际效果为什么差看完-anthropic-这篇博客刷分的因素太单一了|国产顶尖模型 benchmark 评分那么高，可实际效果为什么差？看完 Anthropic 这篇博客，刷分的因素太单一了]]
- [[entities/你写的-skill及格了吗|你写的 Skill，及格了吗？]]
- [[entities/2-小时0-行手写代码我用-claude-做了一个生产级-vscode-插件|2 小时，0 行手写代码，我用 Claude 做了一个生产级 VSCode 插件]]
- [[entities/anthropic-官方-agent-harness-平台claude-managed-agents-完整指南|Anthropic 官方 Agent Harness 平台：Claude Managed Agents 完整指南]]
- [[entities/imclaw通过微信飞书操控claude-code-coodex-gemini-clipi-agent蜂群|IMClaw：通过微信/飞书操控ClaudeCode/Codex/GeminiCLI/Pi Agent蜂群]]
- [[entities/民生银行基于规格驱动开发sdd的-codeagent-私域研发探索与实践|民生银行基于规格驱动开发（SDD）的 CodeAgent 私域研发探索与实践]]
- [[entities/claude-code-20000-char-source-analysis|两万字详解Claude Code源码核心机制]]
- [[entities/天猫新品团队ai编码实战指南下|天猫新品营销技术团队AI编码实战指南（上）]]
- [[entities/深入理解-claude-code-源码中的-agent-harness-构建之道-v2|深入理解 Claude Code 源码中的 Agent Harness 构建之道]]
- [[entities/从vibe-coding到agentic-engineering重构后台开发全流程|从Vibe Coding到Agentic Engineering：重构后台开发全流程]]
- [[entities/ai-agent-engineer-capability-map|AI Agent 工程师能力地图]]