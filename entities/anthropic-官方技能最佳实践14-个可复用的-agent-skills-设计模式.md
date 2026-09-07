---

title: "Anthropic 官方技能最佳实践：14 个可复用的 Agent Skills 设计模式"
type: entity
tags: [agent, skill-design, anthropic, claude-code]
sources: [raw/articles/anthropic-14-skill-patterns-best-practices, raw/articles/anthropic-agent-skills-design-patterns-14, raw/articles/anthropic-官方技能最佳实践14-个可复用的-agent-skills-设计模式]
created: 2026-05-10
updated: 2026-09-07
review_value: 5
review_confidence: 10
review_recommendation: worth-reading
review_stars: 3
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: 14模式第二份重复; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Anthropic 官方技能最佳实践：14 个可复用的 Agent Skills 设计模式

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/anthropic-官方技能最佳实践14-个可复用的-agent-skills-设计模式.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agent-oriented-infra-intent-driven-code-sedimentation|晓斌：从 People-Oriented 到 Agent-Oriented Infra —— 意图驱动 + 代码沉淀的进化体]] — Agent-Oriented Infra长文
- [[entities/harness-engineering-7-layers-openclaw-hermes-claude-code-p1anu|Harness 到底是什么？看看 OpenClaw、Hermes、Claude Code 的演绎吧]] — 三框架演绎七层模型12857字rv9
- [[entities/open-claw-tool-bus-subagent-architecture|800行代码实现 Open Claw 的 Tool、消息总线、子Agent管理架构]] — 薄抽象显式控制流8802字rv9全版
- [[entities/wangyunhe-harness-optimization-agentsoul|王云鹤眼中的Harness：复杂优化问题，AGI灵魂争夺之战]] — Agent=Models+Harness联合优化
- [[entities/claude-发布官方报告承认存在-3-处质量退化问题|Claude 发布官方报告，承认存在 3 处质量退化问题]] — 三重变更叠加复盘
- [[entities/anthropic-llm-attck-navigator-cyber-operations|Anthropic LLM ATT&CK Navigator: AI-Enabled Cyber Operations]] — ARiES风险评分
- [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构|800行代码实现 Open Claw 的 Tool、消息总线、子Agent管理架构]] — agent核心组件实现
- [[entities/gpt-54-is-a-big-step-for-codex|GPT 5.4 是 Codex 的一次大跨越：四维评估视角与 Agent 战争回归]] — 四维评估+Claude/GPT哲学分歧5698字全版
- [[entities/claude-code-extended-thinking-not-authentic|The text in Claude Code’s “Extended Thinking” output is not authentic. – blog]] — thinking签名发现
- [[entities/claude-perceived-degradation-anthropic-effort-model-explanation-2026|全网骂Claude变笨，Anthropic下场揭秘：坑你的不是模型]] — Model vs Effort框架

## 工程实践
- [[entities/boris-cherny-新访谈开发工具正在从-ide-变成-agent-控制台|Boris Cherny 新访谈：开发工具正在从 IDE 变成 Agent 控制台]] — Boris访谈rv10全版
- [[entities/claude-code-core-internals|Claude Code 源码核心机制详解]] — 源码机制18k主版
- [[entities/claude-code-deep-architecture-analysis|Claude Code 架构深度解析]] — 并发与延迟加载深析
- [[entities/claude-code-openclaw-memory-comparison|Claude Code Openclaw Memory Comparison]] — 记忆系统对比rv9
- [[entities/claude-code-source-leak-lifecycle-analysis|CLAUDE.md]] — 8步生命周期10k
- [[entities/tencent-skill-writing-complete-playbook-jackjchou|鹅厂 Skill 写作完整 Playbook：14 章节 end-to-end 实战 + 工程化评估（腾讯一线踩坑 + Anthropic 官方做法整合）]] — 14章节skill写作playbook
- [[entities/anthropic-mcp-revisited-tool-search-code-orchestration|Anthropic 最新博客：MCP 没死，它又来了]] — MCP三条路
- [[entities/harness-design-long-running-apps|长时间运行应用的 Harness 设计]] — GAN式generator/evaluator长时harness 9745字
- [[entities/knowledge-work-plugins-shuge-anthropic-deep-source|knowledge-work-plugins拆解：Anthropic官方开源，4 种组件、3 级加载、2 层记忆，纯文件的 AI岗位插件集]] — 岗位级封装+三级披露+两层记忆7956字
- [[entities/anthropic-12-mcp-production-patterns|Anthropic 官方生产级 Agent 最佳实践：12 个可复用的 MCP 设计模式]] — 12个MCP模式
- [[entities/anthropic-14-skill-patterns-best-practices|Anthropic 官方技能最佳实践：14 个可复用的 Agent Skills 设计模式]] — 14个skill模式全版
- [[entities/claude-code-checkup-feature-boris-cherny-2026|Claude Code /checkup 功能：清理 Skills/MCP 提升性能]] — 配置腐烂治理
- [[entities/anthropic-claude-code-large-scale-code-migration-2026|Anthropic 大规模代码迁移方法论 — Claude Code 多 Agent Loop 的工程实践]] — 迁移六步方法论
- [[entities/claude-code-automatic-feedback-report-self-diagnosis|Claude Code 自动反馈报告功能：AI Agent 的自我诊断与改进机制]] — 反馈报告短条

## 延伸导航
- [[moc/claude-code-complete-guide|Claude Code 生态完全指南]]
