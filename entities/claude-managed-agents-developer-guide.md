---

title: "Claude Managed Agents 开发者指南"
created: 2026-05-01
updated: 2026-09-10
type: entity
tags: [anthropic, managed-agents, harness, cloud-agent, api]
sources:
  - raw/articles/claude-managed-agents-developer-guide
  - raw/articles/anthropic-pm-jess-yan-managed-agents
review_value: 8
review_confidence: 7
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: 开发者指南与平台主版重复; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Claude Managed Agents 开发者指南

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/claude-managed-agents-developer-guide.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agent-oriented-infra-intent-driven-code-sedimentation|晓斌：从 People-Oriented 到 Agent-Oriented Infra —— 意图驱动 + 代码沉淀的进化体]] — Agent-Oriented Infra长文
- [[entities/anthropic-n-days-frontier-agent-vulnerability-research|Anthropic N-days: Frontier Agent Vulnerability Research]] — N-day研究
- [[entities/agent-harness-12-components-7-decisions|一篇看懂 Agent Harness 的结构！ — 12组件+7决策完整框架]] — harness 12组件框架
- [[entities/harness-之后-状态边界与失败闭环-ruofei|'Harness 之后：状态边界与失败闭环（若飞续篇）']] — 运行时契约/提交闸门/失败回写10308字全版
- [[entities/from-prompt-to-harness-claude-official|从 Prompt 到 Harness：Claude 官方学习资料]] — Harness五子系统闭环解读
- [[entities/claude-opus-47|Claude Opus 4.7 并不是一次全面升级，甚至部分能力大幅衰退]] — 4.7衰退面分析
- [[entities/ai-agent-harness-construction-akshay|深度拆解：AI 智能体 Harness 的构造（译）]] — harness构造译全版

## 工程实践
- [[entities/long-running-agent-ralph-loop-handover-harness-ruofei|'长周期 Agent 详解：从 Ralph Loop 到可接管 Harness']] — 三类漂移+5张卡治理12390字rv10全版
- [[entities/loop-engineering-addy-osmani-challengehub|Loop Engineering:不再写提示词,而是设计替你写提示词的循环——先写刹车再写循环（19 来源深度合并：Addy Osmani / Boris Cherny+Peter Steinberger / 教科书 / 若飞 工程现场 / TechFarrari 批判 / 若飞 实用指南 / 爱范儿 科普批判 / AllenTang Karpathy 尺子 / winty 7架构中文主流视角 / AutoResearch 5 决策 / 三层结构 + 三款产品对比 + Ralph Loop + 准备度总表 / Shubham Saboo PM 视角 / 若飞 吴恩达三层Loop）]] — 19来源合并Loop Engineering巨著73742字rv10
- [[entities/boris-cherny-新访谈开发工具正在从-ide-变成-agent-控制台|Boris Cherny 新访谈：开发工具正在从 IDE 变成 Agent 控制台]] — Boris访谈rv10全版
- [[entities/karpathy-vibe-coding-agentic-engineering-v4|Karpathy 最新访谈：从 Vibe Coding 到 Agentic Engineering]] — v4 8090字rv10：可验证性上限+MenuGen警示
- [[entities/mac-multi-agent-coding-skills-hooks-harness|MAC（multi-agent-coding）：Skills + Hooks 两层 Harness —— 完全委托 0-20% 的解法]] — Skills概率层+Hooks确定性层两层Harness
- [[entities/anthropic-claude-managed-agents-platform-2026|Anthropic Claude Managed Agents 平台正式发布]] — 平台发布14k主版
- [[entities/anthropic-demystifying-evals-for-ai-agents|Anthropic Demystifying Evals for AI Agents]] — eval概念框架
- [[entities/claude-code-routines-proactive-agent|Claude Code Routines：从工具到队友的主动 Agent 模式]] — Routines三能力
- [[entities/anthropic-computer-use-best-practices|Anthropic 发布 Computer Use 最佳实践]] — 坐标映射与梯度分配
- [[entities/anthropic-claude-managed-agents-guide|Claude Managed Agents 官方 Harness 平台指南]] — 平台指南分析版
- [[entities/anthropic-managed-agents-scaling|Anthropic Managed Agents：用 K8s 思路虚拟化 Agent 组件]] — 宠物到牛群
- [[entities/长周期-agent-详解-从-ralph-loop-到可接管-harness|长周期-agent-详解-从-ralph-loop-到可接管-harness]] — Ralph loop到接管harness
- [[entities/anthropic-pm-jess-yan-managed-agents|Anthropic PM Jess Yan 的三个 Claude Agent 实践]] — PM三agent实践
- [[entities/claude-code-seven-customization-methods-anthropic-official|Claude Code 七种自定义方法：官方全景指南]] — 七种自定义对比
- [[entities/prompt-debugger-compare-templates-winty|Prompt 调试器：A/B 测试模板对比]] — Prompt调试器三件套：A/B+评分沉淀+模板库
- [[entities/专为-managed-agents-而生的-harness-底座agentscope-20|专为 Managed Agents 而生的 Harness 底座：AgentScope 2.0]] — brain hands拆分控制面数据面
- [[entities/ai-native-sdlc-playbook-anthropic|AI Native SDLC Playbook：Anthropic 应用 AI 团队的软件开发生命周期重构方法论]] — SDLC六阶段重构

## 关联

- 同题异语种孪生页：[[entities/anthropic-官方-agent-harness-平台claude-managed-agents-完整指南]]（归并候选，提案卡 #11 批1）
- 同题异语种孪生页：[[entities/深入理解-claude-code-源码中的-agent-harness-构建之道-v2]]（归并候选，提案卡 #11 批1）

