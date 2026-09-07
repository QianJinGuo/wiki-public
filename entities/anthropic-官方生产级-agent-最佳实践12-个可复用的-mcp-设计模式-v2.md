---
title: "Anthropic 官方生产级 Agent 最佳实践：12 个可复用的 MCP 设计模式"
created: 2026-06-11
updated: 2026-09-07
type: entity
tags: [mcp, anthropic, agent-design-patterns, production, best-practices, tool-integration, authentication, context-economy, plugin]
sources: [raw/articles/anthropic-官方生产级-agent-最佳实践12-个可复用的-mcp-设计模式-v2]
provenance_state: extracted
confidence: 0.9
review_value: 5
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: 12模式重复v2; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Anthropic 官方生产级 Agent 最佳实践：12 个可复用的 MCP 设计模式

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/anthropic-官方生产级-agent-最佳实践12-个可复用的-mcp-设计模式-v2.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agent-oriented-infra-intent-driven-code-sedimentation|晓斌：从 People-Oriented 到 Agent-Oriented Infra —— 意图驱动 + 代码沉淀的进化体]] — Agent-Oriented Infra长文
- [[entities/anthropic-ai-windows-mcp-strategy-geekpark-2026|OpenAI 的最强对手，离「AI Windows」又近了一步]] — MCP战略分析
- [[entities/the-new-ai-lock-in|The new AI lock-in]] — 锁定层级迁移分析

## 工程实践
- [[entities/loop-engineering-addy-osmani-challengehub|Loop Engineering:不再写提示词,而是设计替你写提示词的循环——先写刹车再写循环（19 来源深度合并：Addy Osmani / Boris Cherny+Peter Steinberger / 教科书 / 若飞 工程现场 / TechFarrari 批判 / 若飞 实用指南 / 爱范儿 科普批判 / AllenTang Karpathy 尺子 / winty 7架构中文主流视角 / AutoResearch 5 决策 / 三层结构 + 三款产品对比 + Ralph Loop + 准备度总表 / Shubham Saboo PM 视角 / 若飞 吴恩达三层Loop）]] — 19来源合并Loop Engineering巨著73742字rv10
- [[entities/boris-cherny-新访谈开发工具正在从-ide-变成-agent-控制台|Boris Cherny 新访谈：开发工具正在从 IDE 变成 Agent 控制台]] — Boris访谈rv10全版
- [[entities/yumanju-ai-full-flow-efficiency|柚漫剧 AI 全流程提效拆解]] — rv10全流程提效规则基建
- [[entities/claude-code-source-deep-dive-warrior|Claude Code 源码深度解析（13 核心机制）]] — 13机制rv10
- [[entities/claude-code-core-internals|Claude Code 源码核心机制详解]] — 源码机制18k主版
- [[entities/claude-code-skills-practical-guide-discovery-frontmatter|Claude Code Skills 实战指南 — 发现机制、编写与安全]] — 发现机制与安全
- [[entities/gaode-saojie-image-selection-hermesagent-vlm-production-2026|高德扫街榜 HermesAgent 配图系统：VLM + Skill + 语言驱动的生产级 Agent 架构]] — 确定性流水线+Agent巧活，提效48倍rv9
- [[entities/claude-code-skills-mcp-rules-source-analysis|Claude Code Skills / MCP / Rules 源码分析]] — 三个注入位置
- [[entities/claude-managed-agents-self-hosted-sandbox-mcp-tunnels-enterprise|Claude Managed Agents 新更新\]] — brain/hands分离全版
- [[entities/claude-code-mcp-server|Claude Code MCP Server]] — MCP实现机制
- [[entities/tencent-skill-writing-complete-playbook-jackjchou|鹅厂 Skill 写作完整 Playbook：14 章节 end-to-end 实战 + 工程化评估（腾讯一线踩坑 + Anthropic 官方做法整合）]] — 14章节skill写作playbook
- [[entities/anthropic-mcp-revisited-tool-search-code-orchestration|Anthropic 最新博客：MCP 没死，它又来了]] — MCP三条路
- [[entities/production-ai-agents-mcp-cli-skills-stack-ayi|如何构建生产准备的AI代理：MCP、CLI与技能——适合合适的工作的工具]] — 三层连接栈互补：Skill/CLI/MCP，MCP token开销解法
- [[entities/knowledge-work-plugins-shuge-anthropic-deep-source|knowledge-work-plugins拆解：Anthropic官方开源，4 种组件、3 级加载、2 层记忆，纯文件的 AI岗位插件集]] — 岗位级封装+三级披露+两层记忆7956字
- [[entities/gaode-voc-hermes-multi-agent-auto-triage-2026|高德交易 VOC 自动排查：基于 Hermes 的多 Agent 架构实践]] — VOC主从分离多Agent，86%诊断准确率
- [[entities/anthropic-12-mcp-production-patterns|Anthropic 官方生产级 Agent 最佳实践：12 个可复用的 MCP 设计模式]] — 12个MCP模式
- [[entities/erik-schluntz-vibe-coding-in-production|Vibe Coding in Production — Erik Schluntz / Anthropic]] — 验证抽象层方法论，短但有独立洞见
- [[entities/anthropic-claude-skill-9-categories-datawhale-2026|Anthropic Claude Skill 9 类任务分类法]] — 9类分类法
- [[entities/agent-protocol-cost-evolution-roundtable-2026|Agent落地真相：协议、成本与进化——一场关于智能体从能跑通到能投产的讨论]] — 落地圆桌三缺口
- [[entities/agent-skills-development-guide|Agent Skills 开发指南：6 字段规范、3 级加载、5 步评估闭环]] — 6字段开发指南
- [[entities/claude-cowork-2026-big-update|Claude Cowork 大更新：彻夜自动编程的新时代]] — 云端持久化

## 延伸导航
- [[moc/claude-code-complete-guide|Claude Code 生态完全指南]]
- [[moc/mcp-server-patterns|MCP 协议在实际生产中的主要局限是什么？]]
