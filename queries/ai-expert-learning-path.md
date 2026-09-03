---
title: AI 领域专家学习路径
type: query
tags: [learning, ai, expert, roadmap, agent, harness-engineering, interview]
created: 2026-05-17
updated: 2026-05-17
status: active
priority: p0
---
# 如何成为 AI Agent 工程专家并获得大厂 Offer？
> 目标：有开发基础 → 成为 AI Agent 工程专家，能拿大厂 AI infra / Agent engineering / AI platform 方向 offer。
> 核心教材：本知识库 2378 页，主线是 [[queries/harness-engineering-core-principles-best-practices|Agent Harness Engineering]]、Claude Code Ecosystem Topic Map（已删除）、[[moc/ai-skill-design|AI Skill 设计]]、[[moc/agent-memory-architecture|Agent 记忆架构]]。
## 严厉导师规则
1. 每天必须产出 1 份学习记录，写入 ：3 个事实、2 个判断、1 个可复用设计原则。
2. 每周必须交付 1 个可运行项目、1 张架构图、1 次 30 分钟口述复盘。
3. 不接受“看过了”。验收标准只有三类：能复述、能设计、能实现。
4. 学习顺序不按新闻热度走，按能力栈走：模型基础 → Agent loop → Harness → Memory / Context → Skill / MCP → Eval / Observability → Security → Enterprise platform → 面试表达。
5. 每天最多 3 篇主读材料。读不完说明你没有提炼能力，不是材料太多。
## 毕业能力模型
| 能力 | 毕业标准 | 代表材料 |
|---|---|---|
| 模型基本功 | 解释 Transformer、Attention、Scaling Laws、SFT/RLHF/RLAIF 与推理成本 | [[concepts/transformer-architecture]]、[[concepts/attention-mechanism]]、[[concepts/scaling-laws]] |
| Agent 工程 | 从零设计单 Agent loop、工具协议、错误恢复、成本控制 | 、 |
| Harness Engineering | 能区分 Prompt / Context / Harness，设计硬约束与完成门禁 | [[concepts/harness-engineering-framework]]、[[queries/harness-minimum-checklist]] |
| Context / Memory | 设计工作记忆、长期记忆、压缩、召回、冲突解决 | [[concepts/agent-memory-system-design]]、 |
| Skill / MCP | 能写 Skill，能解释 CLI / MCP / API 选型和 token 成本 | [[concepts/hermes-agent-skill]]、[[anthropic-12-mcp-production-patterns]] |
| Multi-Agent | 能设计 Orchestrator-Worker、隔离协议、上下文卫生 |  |
| Production | 能讲清安全、可观测性、治理、成本、权限、审计 | [[concepts/agent-security-full-lifecycle-system]] |
| 面试表达 | 形成 8 个 STAR 故事、3 个系统设计方案、1 个作品集仓库 |
## 每日打卡模板
```md
## Day N
- 主读：
- 概念：
- 设计/项目：
- 今日输出：
- 自测分数：复述 / 设计 / 实现，各 0-10
- 卡点：
```
## 30 天冲刺路线
### Week 1：模型与 Agent 入门，不许跳过基本功
| Day | 主读材料 | 必懂概念 | 当日交付 |
|---|---|---|---|
| 1 | [[concepts/transformer-architecture]]、[[concepts/attention-mechanism]] | token、embedding、self-attention、decoder-only | 手画 Transformer 数据流，写 500 字解释“为什么 attention 适合工具调用前的上下文聚合” |
| 2 | [[concepts/scaling-laws]]、[[concepts/llm-pretraining-vs-sft]] | compute optimal、pretraining、SFT、分布漂移 | 用表格比较 GPT 类模型训练阶段；列 5 个 Agent 场景下 SFT 不足 |
| 3 |  | vibe coding、agentic engineering、human-in-the-loop | 写"从开发者到 AI 任务导演"的能力差距清单 |
| 4 |  | agent loop、plan-act-observe、tool calling | 实现一个 3 工具 toy agent：search/read/write，用结构化错误回传 |
| 5 |  | async、timeout、function schema、工具失败 | 给 Day 4 agent 加 timeout、retry、错误分类 |
| 6 |  | context window、working set、context rot | 画出"用户目标→上下文选择→工具调用→验证"的上下文流图 |
| 7 | 面试三层能力模型、A/B/C/D 档 | 复盘 Week 1：录 10 分钟口述，回答“Agent 为什么不是 API wrapper？” |
### Week 2：Harness Engineering 核心
| Day | 主读材料 | 必懂概念 | 当日交付 |
|---|---|---|---|
| 8 | [[concepts/harness-engineering-framework]]、 | Prompt / Context / Harness、软约束 vs 硬约束 | 写一页“Harness 定义”，必须能反驳“只是提示词工程” |
| 9 | [[concepts/harness-engineering-7-layers-framework]] | 七层框架、控制回路、验证门禁 | 把 Day 4 agent 映射到七层框架，指出缺失层 |
| 10 |  | 12 组件、7 决策、模式选择 | 做一个 harness architecture decision record |
| 11 | [[queries/harness-minimum-checklist]] | AGENTS.md、feature list、progress file、completion gate | 给 toy agent 加 `feature_list.json`、progress log、completion check |
| 13 | [[concepts/harness-component-expiry-and-build-to-delete]] | 模型-壳匹配、组件保质期、ablation | 对 Day 11 harness 做 2 个 ablation：移除 retry / 移除 gate |
### Week 3：Memory、Context 与 Skill
| Day | 主读材料 | 必懂概念 | 当日交付 |
|---|---|---|---|
| 15 | [[concepts/agent-memory-system-design]]、 | 工作记忆、情景记忆、语义记忆、程序性记忆 | 设计一个三层 memory schema |
| 16 | [[concepts/agent-memory-systematic-framework]]、 | memory lifecycle、write policy、conflict resolution | 实现 SQLite 记忆表：facts / episodes / skills |
| 17 |  | session memory、project memory、vector DB 争议 | 写"什么时候不该用向量数据库"的判断标准 |
| 18 | [[concepts/hermes-agent-skill]]、 | skill as procedural memory、经验沉淀 | 把 Day 11 的完成门禁封装成一个可复用 Skill |
| 19 | [[comparisons/skill-system-design-comparison]]、 | Skill 设计模式、触发条件、边界 | 写 3 个 Skills：读代码、写测试、生成复盘 |
| 20 | [[anthropic-12-mcp-production-patterns]] | MCP、tool registry、tool search、权限边界 | 设计 CLI / MCP / API 三选一决策表 |
| 21 | 、 | token efficiency、tool search、工具压缩 | Week 3 验收：让 agent 自动选择 skill，并记录 token / latency |
### Week 4：Multi-Agent、评测、可观测性、安全
| Day | 主读材料 | 必懂概念 | 当日交付 |
|---|---|---|---|
| 22 |  | Orchestrator-Worker、上下文隔离、fan-out | 把 toy agent 拆成 planner / executor / verifier |
| 23 | 、 | context hygiene、摘要回传、隔离协议 | 设计子 Agent 交接协议：输入、输出、禁止事项 |
| 26 | [[concepts/agent-security-full-lifecycle-system]]、 | tool poisoning、least privilege、sandbox | 写 threat model：资产、攻击面、缓解策略 |
| 27 | 、[[concepts/openclaw-architecture]] | gateway、runtime、memory、skill、governance | 画 OpenClaw / Claude Code / Hermes 对比架构图 |
| 28 | 、 | Claude Code 核心机制、action space、agent view | 复述 13 个核心机制，挑 3 个迁移到你的 harness |
| 29 | 、 | production harness、component tradeoff | 写大厂面试系统设计题：生产级 Coding Agent 平台 |
| 30 |  | STAR、系统设计、踩坑故事 | 30 天总验收：20 分钟模拟面试 + 作品集 README 初稿 |
## 100 天完整路线
### Days 31-40：Claude Code / OpenClaw / Hermes 深拆
| Day | 学习任务 | 设计/项目 |
|---|---|---|
| 31 | Claude Code Ecosystem Topic Map（已删除）、 | 画 Claude Code 模块图 |
| 32 | 、 | 写 prompt/context/harness 三层对照表 |
| 33 | 、[[concepts/claude-code-tool-design-evolution]] | 重新设计你的工具 schema |
| 34 | 、 | 实现上下文压缩策略 v1 |
| 35 | 、 | 写 Skills / MCP / Rules 区分文档 |
| 36 | [[moc/openclaw-architecture]]、 | 画 OpenClaw 五层架构 |
| 38 | [[queries/hermes-agent-core-architecture-self-evolution]] | [[concepts/hermes-agent]] | 画 Hermes 四模块图 |
| 39 |  | 设计 skill 自动沉淀流程 |
| 40 |  | 交付三框架 3000 字横评 |
### Days 41-50：RAG、检索、知识库与第二大脑
| Day | 学习任务 | 设计/项目 |
|---|---|---|
| 41 | [[moc/rag-knowledge-retrieval]]、 | 比较 classic / graph / agentic RAG |
| 42 |  | 设计 wiki 检索 pipeline |
| 43 | 、 | 写 agentic search 方案 |
| 44 | 、 | 设计个人知识库 agent |
| 45 | 、[[concepts/ai-team-knowledge-harness]] | 写团队知识沉淀 SOP |
| 46 | [[queries/wiki-capability-self-assessment]]、 | 给本 wiki 做学习视角健康检查 |
| 47 | [[moc/wiki-master-map]]、[[queries/research-frontier-map]] | 生成个人研究地图 |
| 48 | 、 | 实验 Skill-RAG vs 普通 RAG |
| 49 |  | 实现 working set 选择器 |
| 50 | 自选 3 篇相关 raw source | 交付“第二大脑学习 Agent”PRD |
### Days 51-60：企业级 Agent 平台与云原生落地
| Day | 学习任务 | 设计/项目 |
|---|---|---|
| 51 | AI Agent Platforms Topic Map（已删除）、 | 设计企业搜索 Agent |
| 52 | 、 | 画 AgentCore runtime 架构 |
| 53 | 、 | 设计企业 Agent 身份与权限 |
| 54 | 、 | 设计质量优化飞轮 |
| 55 | 、 | 设计多租户部署架构 |
| 56 | 、 | 拆 2 个行业案例 |
| 57 | 、 | 写治理清单 |
| 58 | 、 | 设计 agent-native data layer |
| 59 | 、[[concepts/openai-realtime-voice-architecture]] | 设计实时语音 Agent |
| 60 | Week 8 复盘 | 交付企业级 Agent 平台系统设计文档 |
### Days 61-70：安全、可靠性、成本和可观测性强化
| Day | 学习任务 | 设计/项目 |
|---|---|---|
| 61 | [[queries/ai-agent-security-threat-vectors-mitigation]]、 | 写 agent security test plan |
| 62 | 、 | 设计红队 agent |
| 63 | 、 | 给 Skills 做安全扫描规则 |
| 64 | 、 | 写可靠性失效模式库 |
| 65 | 、 | 做 token / cache 成本优化 |
| 69 | [[queries/harness-peer-review-framework]]、[[queries/web-content-reviewer-evaluation]] | 做一次对抗式 peer review |
| 70 | Week 10 复盘 | 交付“可靠性与安全白皮书” |
### Days 71-80：前沿论文与研究判断
| Day | 学习任务 | 设计/项目 |
|---|---|---|
| 71 | [[queries/ai-model-research-latest-directions]]、[[queries/llm-training-rl-research]] | 建论文阅读评分表 |
| 72 | 、 | 讲清 RL without reward model |
| 73 | 、 | 解释 trajectory modeling 对 agent eval 的意义 |
| 75 | 、 | 写可解释性短评 |
| 76 | [[concepts/mechanistic-interpretability]]、 | 讲 SAE 和 NLA 的区别 |
| 77 | 、[[concepts/skill-formal-theory-survey]] | 整理 self-evolving agent taxonomy |
| 78 | 、 | 设计 skill curation 算法 |
| 79 | 、 | 写 harness evolution 综述 |
| 80 | Week 12 复盘 | 交付 1 篇可公开技术文章草稿 |
### Days 81-90：作品集冲刺
| Day | 学习任务 | 设计/项目 |
|---|---|---|
| 81 | 回看 [[queries/harness-minimum-checklist]] | 项目 1：生产级 Coding Agent Harness 立项 |
| 83 | 回看 [[concepts/hermes-agent-skill]] | 实现 Skills 注册与触发 |
| 84 | 回看 [[anthropic-12-mcp-production-patterns]] | 接入 1 个真实工具或 MCP |
| 85 | 回看 [[comparisons/agent-skill-evaluation-methods]] | 加 eval suite |
| 87 | 回看 [[concepts/agent-security-full-lifecycle-system]] | 加权限和安全策略 |
| 89 | 回看  | 写系统设计文档 |
| 90 | 项目 1 完成 | 录制 5 分钟 demo，README 写清架构与 tradeoff |
### Days 91-100：大厂面试冲刺
| Day | 学习任务 | 设计/项目 |
|---|---|---|
| 91 | 整理 8 个 STAR 故事 |
| 92 |  | 准备 30 个高频问答 |
| 93 | 、 | 准备架构师视角回答 |
| 94 | 、 | 准备业务与护城河视角 |
| 95 | 系统设计 1 | 设计“企业知识库 Agent 平台” |
| 96 | 系统设计 2 | 设计“AI coding platform for 1000 engineers” |
| 97 | 系统设计 3 | 设计“安全可审计 MCP gateway” |
| 98 | 模拟面试 1 | 45 分钟技术 + 15 分钟复盘 |
| 99 | 模拟面试 2 | 45 分钟系统设计 + 15 分钟复盘 |
| 100 | 终局答辩 | 30 分钟讲作品集，10 分钟讲研究判断，10 分钟讲职业定位 |
## 每周固定作业
| 周 | 项目 | 验收 |
|---|---|---|
| 1 | 3 工具单 Agent | 工具失败可恢复，日志可读 |
| 2 | 最小 Harness | 有 feature list、progress、completion gate、eval |
| 3 | Memory + Skill Agent | 能写入/召回记忆，能触发 Skill |
| 4 | Multi-Agent Harness | planner / executor / verifier 隔离清楚 |
| 5 | 三框架横评 | Claude Code / OpenClaw / Hermes 能讲清 tradeoff |
| 6 | 第二大脑学习 Agent PRD | 能说明检索、学习、复盘、打卡闭环 |
| 7-8 | 企业级 Agent 平台设计 | 多租户、权限、可观测性、成本、治理完整 |
| 9-10 | 安全可靠性白皮书 | threat model、eval、审计、红队流程完整 |
| 11-12 | 前沿研究综述 | 至少 10 篇材料，形成原创判断 |
| 13 | 作品集项目 | README、demo、架构图、eval 报告齐全 |
| 14 | 面试冲刺 | 8 个故事、3 个系统设计、2 次模拟面试 |
## 大厂 Offer 作品集标准
必须准备 1 个主项目和 2 个副项目：
1. 主项目：Production Coding Agent Harness
   - 核心模块：context manager、tool registry、skill runtime、memory、eval、observability、security policy。
   - README 必须说明：为什么不是简单 chatbot，为什么不盲目 multi-agent，如何做可靠性闭环。
2. 副项目 A：Second Brain Learning Agent
   - 基于本 wiki，完成学习路径推荐、每日打卡、薄弱点诊断、复习调度。
3. 副项目 B：MCP Gateway / Skill Security Scanner
   - 检测 tool poisoning、过宽权限、token 爆炸、缺失审计。
## 面试题库
### 必会解释题
- Agent loop 和传统 workflow 的区别是什么？
- 为什么 Harness 是硬约束，而 prompt 是软约束？
- Function calling 的失败类型有哪些？如何结构化回传？
- MCP 的 token 成本为什么可能很高？什么时候用 CLI 更好？
- Memory 与 RAG 的区别是什么？什么时候不用向量库？
- Multi-Agent 为什么会放大幻觉？如何隔离上下文？
- Pass@k 和 Pass^k 分别评估什么？
- 生产级 Agent 的可观测性至少记录哪些字段？
- Tool poisoning 如何发生？如何防御？
- 如何证明一个 Agent 改版真的更好，而不是只是 benchmark 变好？
### 必会系统设计题
- 设计一个企业内部知识库问答 Agent。
- 设计一个能长期执行代码任务的 Coding Agent Harness。
- 设计一个多租户 MCP Gateway。
- 设计一个 Agent 评测平台。
- 设计一个带长期记忆的个人 AI 助理。
- 设计一个 1000 人研发团队的 AI coding 治理体系。
## 评分标准
| 分数 | 状态 |
|---|---|
| 0-59 | 仍是 API 调用者，不能投大厂 AI 岗 |
| 60-74 | 可做 AI 应用工程师，但系统设计薄弱 |
| 75-84 | 可投 Agent 工程 / AI 平台岗位 |
| 85-92 | 具备大厂强竞争力 |
| 93-100 | 具备专家候选人水平，能输出原创框架 |
每 10 天自评一次：模型基础 10、Agent loop 10、Harness 15、Context/Memory 15、Skill/MCP 10、Eval 15、安全/可观测 10、系统设计 10、表达 5。
## 第一学期最终答辩
答辩题：**“为什么 2026 年 AI 工程师的核心能力不是写 prompt，而是设计可验证、可治理、可进化的 Agent Harness？”**
合格答案必须同时覆盖：
- 模型能力边界；
- 工具与上下文边界；
- 可靠性与评测；
- 记忆与技能沉淀；
- 组织级治理；
- 成本、安全和可观测性；
- 个人作品集中的实证数据。
## 相关实体
