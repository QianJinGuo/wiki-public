---
title: "Agent Engineering 能力地图"
created: 2026-06-12
updated: 2026-08-01
type: concept
tags: [concept, agent-engineering, capability, skill, career, engineer]
sources: [entities/ai-agent-engineer-capability-map, entities/agent-engineering-principles-architecture-practice]
---

## 定义

Agent Engineering 能力地图：建一个 production 级 agent 系统需要的 7 大能力域（context engineering / harness 设计 / memory 架构 / tool 设计 / verifier / orchestration / observability）。和 ML engineering 的对比：少了训练管线，多了 prompt + tool design + 实时调优。

## 核心范式

- **context engineering 是基本功**：working set 怎么装、怎么裁、怎么 swap
- **harness 设计决定上限**：loop / state / tool catalog / governance 全栈
- **memory + tool 是杠杆**：好的 memory schema + tool API 让模型表现翻倍
- **observability 是 production gate**：没有可观测性，agent 在线就是黑盒赌博

## 背景与提出

Agent Engineering 作为独立职能在 2026 年成型——它不是 ML Engineering 的子集，也不是 Software Engineering 的延伸，而是一个需要独特技能栈的新领域。^[entities/ai-agent-engineer-capability-map] 能力地图的提出源于一个现实：很多团队用 ML Engineer 的 JD 招 Agent Engineer，或者让 Backend Engineer 兼做 Agent 开发——结果两边都不专业。Agent Engineering 有自己的核心能力域，需要显式定义。

## 范式细节

Agent Engineering 的能力地图分 6 个域。上下文工程：working set 管理、prompt 设计、memory 策略、context 裁剪。这是最独特的域——传统 SE 没有「管理概率性上下文」的需求。Harness 设计：agent 主循环架构、工具设计、错误恢复、状态管理。最接近传统 SE 但多了概率层。Verifier 设计：测试策略、LLM-as-judge、安全扫描、人类 gate。传统测试的升级版，需要处理开放式输出。可观测性：trace/spans/metrics/logging，agent 行为的可解释性。部署运维：cron 调度、多 agent 编排、成本控制、灰度。领域知识：对具体业务场景的理解，决定 agent 的上限。6 个域中，上下文工程和 verifier 设计是 Agent Engineering 最独特的——其他 4 个在传统 SE 中有对应但范式不同。^[entities/agent-engineering-principles-architecture-practice]

具体而言，[[entities/agent-context-management-architecture-patterns|上下文管理架构模式]] 覆盖了 working set 的裁剪策略与优先级调度；[[entities/harness-engineering-framework|Harness Engineering Framework]] 定义了 agent 主循环的标准组件与状态迁移规则；[[entities/llm-as-a-verifier-framework|LLM-as-Verifier 框架]] 提供了开放式输出的自动评估方法论；[[entities/agent-harness-observability-production|生产级可观测性方案]] 则解决了 agent 行为的 trace 与 metrics 采集问题。[[entities/agent-orchestration|多 Agent 编排]] 是部署运维域的核心能力，涉及任务分解与结果聚合。[[entities/agent-memory-architecture-essence|Memory 架构本质]] 揭示了如何设计有效的 memory schema 使模型表现翻倍。

## 局限与反对声音

能力地图的一个局限是「过度细分」：6 个域的边界在实践中很模糊——上下文工程和 harness 设计经常交织在一起（working set 是 context 问题还是 harness 问题？）。另一个问题是「领域权重因场景而异」：做 coding agent 的团队，上下文工程和 harness 设计权重最高；做 RAG agent 的团队，领域知识和 verifier 权重最高。统一的能力地图可能掩盖这些差异。还有批评指出：这个地图只覆盖了「工程」没覆盖「研究」——agent 的 prompt 策略、memory 机制本身也在快速演进，需要研究能力。

## 现实案例

Hermes Agent 的 skill 系统按能力域自然分组：上下文工程 → skill frontmatter + context injection；harness 设计 → skill body（步骤/工具/错误处理）；verifier → pre-commit gate + lint；可观测性 → log.md + cron-report；部署运维 → cronjob + profile。一个完整的 skill 开发流程正好走完能力地图的 6 个域，说明这个分类和工程实践是对齐的。^[entities/hermes-agent]

## 现实案例

Agent Engineering 能力地图在 2026 年的实践中已经分化出三个流派。OpenAI / Anthropic 内部：每个能力域有一个专门的工程师团队——context team / harness team / verifier team / observability team。优点是深，缺点是跨域协调成本高。创业公司：一人多能，1 个 agent engineer 跨 6 个域都用工程化产品（LangSmith / Langfuse / Helicone）替代自研。优点是交付快，缺点是天花板低——产品迭代速度跟不上开源框架。开源 Agent 框架（LangChain / LlamaIndex / Hermes Agent）：能力地图不是写在文档里，而是写在代码里——harness 模块、memory 模块、tool 模块各自独立，工程师按需组合。Hermes Agent 的 cronjob + skill + memory 注入是把能力地图做成 runtime 的代表，每个域都有具体的工具实现而非抽象指南。^[entities/hermes-agent]

## 实践启示

在 2026 年搭建 agent 团队时，能力地图可以直接转化为岗位 JD 和招聘评估。JD 拆解：把 6 个能力域作为 6 个评估维度，每个维度 1-2 道面试题——「设计 working set 裁剪策略」对应 context engineering，「为一个 RAG agent 设计 verifier」对应 verifier 设计，「解释为什么 LangChain 的 callback 不能跨 session 持久化」对应 harness 设计。能力地图也帮助识别团队缺口：如果一个 10 人 agent 团队没有 1 个人专精 verifier 设计，那 production gate 就是形式主义。给现有团队做能力地图 audit 时，最容易暴露的缺口是 observability 和 verifier——大多数团队投入大量资源在 prompt 调优和 harness 设计上，但调试 agent 在 production 出问题时没有可观测数据可看。每个能力域都需要至少 1 个「负责人」和 1 个「评估标准」，否则能力地图只是文档摆设。团队规模 < 5 时可以一人多能，但 6 个能力域必须每周有固定时间投入，不能被业务压力挤压为零。

在实践层面，[[entities/agent-skill-writing|Agent Skill 写作方法论]] 为 context engineering 和 tool design 提供了可操作的 skill 封装规范；[[entities/minimax-agent-team-mavis|MiniMax Agent Team 的 Owner-Worker-Verifier 架构]] 展示了如何在团队层面实践 verifier 设计域；[[entities/agent-memory-injection-5-dimensions-4-papers-agent-shouji-2026|Memory 注入的五维四论文框架]] 则深化了 memory 策略的理论基础。

## 与相邻概念的区分

能力地图和「MLE（Machine Learning Engineer）能力图」的区别：MLE 核心是数据 + 模型 + 训练 pipeline + serving infra；Agent Engineer 核心是 context + harness + verifier + observability。两个领域有重叠（部署、监控），但核心技能栈几乎不互通。和「Platform Engineer 能力图」的区别：Platform Engineer 关注底层 infra（K8s / network / storage）；Agent Engineer 关注 LLM 应用层。和「Prompt Engineer 能力图」的区别：Prompt Engineer 只覆盖 6 个能力域中的 1 个（context engineering），把 prompt engineer 当 agent engineer 是团队配置错误的常见信号。能力地图也不同于「agent 框架的功能清单」——框架功能是工具，能力地图是工程师需要掌握的技能。一个用 LangChain 的工程师可能掌握了 5/6 个能力域，也可能是 1/6（只懂 LangChain API 调用）。判断标准是「这个工程师能否在不依赖特定框架的情况下从零搭一个 production agent」——能则是 5/6+，不能则是 1/6。能力地图的价值就在于此：把「会用什么框架」转化为「能解决什么问题」。

[[entities/context-engineering-three-memory-paradigms|上下文工程与三种 Memory 范式]] 的对比进一步厘清了 context engineering 与传统 SE 的本质差异——后者没有「管理概率性上下文」的需求；[[entities/agent-tools-research|Agent Tool 研究]] 则揭示了 tool design 域的深度远超简单 API 调用。

## 能力地图的演进方向

2026 年的能力地图仍然在快速演化。最可能的演进方向：领域特定能力地图分化（coding agent 能力地图 vs RAG agent 能力地图 vs computer-use agent 能力地图）；自动化评估工具普及（自动评估候选人在 6 个域的熟练度）；能力地图和 JD 系统打通（LinkedIn / Greenhouse 直接用能力地图筛简历）。一个反方向的演进可能：能力地图被简化到 3-4 个核心域（context / harness / verifier / observability），因为很多域其实是 context 和 harness 的子能力，不需要单独列。Hermes Agent 的实践显示「4 大能力域」足够覆盖大部分场景——context engineering / harness engineering / verifier engineering / observability engineering。其他域（memory / tool / orchestration / domain）是这 4 大域的子能力，不需要单独招聘。能力地图的简化版更容易在团队内推广——6 个域让 HR 困惑，4 个域可以一眼记住。

## 能力地图的常见误读

能力地图不是「agent engineer 必须精通 6 个域」——那是 senior agent architect 的画像。junior agent engineer 可能在 2-3 个域有深度，其他域靠工具和同事支援。能力地图的真实价值是「诊断」——发现团队的薄弱环节并针对性补强。一个 6/6 团队是理想但罕见，2-3/6 团队是常态。能力地图也帮助新人 onboarding——明确告诉他需要学什么、达到什么标准、谁来评估。

## 在 wiki 中的关联

- [[entities/ai-agent-engineer-capability-map|Agent engineer 能力地图源]]
- [[entities/agent-engineering-principles-architecture-practice|Agent engineering 原理与实践]]
- [[concepts/context-engineering|context engineering]]
- [[concepts/harness-engineering-framework|harness engineering framework]]
- [[concepts/production-agent-engineering|production agent engineering]]

## 进一步阅读

- [[entities/ai-agent-engineer-capability-map]]
- [[entities/agent-engineering-principles-architecture-practice]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
