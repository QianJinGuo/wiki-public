---

title: "Anthropic 博客：Claude Code 大型代码库最佳实践"
type: entity
tags: [agent, anthropic, claude, rag, claude-code, large-codebase, harness, hook, skills, plugins, lsp, mcp, sub-agents, configuration, enterprise-deployment, 治理]
created: 2026-05-21
updated: 2026-09-10
review_value: 9
review_confidence: 9
sources: [raw/articles/anthropic-claude-code-large-codebase-best-practices-50002a089323]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: dup-correction: 同主题保留更全版本; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Anthropic 博客：Claude Code 大型代码库最佳实践

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/anthropic-claude-code-large-codebase-best-practices-50002a089323.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agent-oriented-infra-intent-driven-code-sedimentation|晓斌：从 People-Oriented 到 Agent-Oriented Infra —— 意图驱动 + 代码沉淀的进化体]] — Agent-Oriented Infra长文
- [[entities/anthropic-n-days-frontier-agent-vulnerability-research|Anthropic N-days: Frontier Agent Vulnerability Research]] — N-day研究
- [[entities/wangyunhe-harness-optimization-agentsoul|王云鹤眼中的Harness：复杂优化问题，AGI灵魂争夺之战]] — Agent=Models+Harness联合优化
- [[entities/agentmemory-source-analysis-coding-agent-local-memory|AgentMemory 源码分析：给 Coding Agent 装上本地长期记忆]] — 源码级解析互补
- [[entities/anthropic-llm-attck-navigator-cyber-operations|Anthropic LLM ATT&CK Navigator: AI-Enabled Cyber Operations]] — ARiES风险评分
- [[entities/from-prompt-to-harness-claude-official|从 Prompt 到 Harness：Claude 官方学习资料]] — Harness五子系统闭环解读
- [[entities/claude-code-and-what-comes-next|Claude Code and What Comes Next]] — 压缩/Skills/Subagents
- [[entities/mcp-tool-design-tradeoffs-anthropic-2026|MCP tool design: Practical approaches and tradeoffs]] — 六种MCP工具设计策略V1-V6
- [[entities/claude-perceived-degradation-anthropic-effort-model-explanation-2026|全网骂Claude变笨，Anthropic下场揭秘：坑你的不是模型]] — Model vs Effort框架

## 工程实践
- [[entities/yumanju-ai-full-flow-efficiency|柚漫剧 AI 全流程提效拆解]] — rv10全流程提效规则基建
- [[entities/claude-code-core-internals|Claude Code 源码核心机制详解]] — 源码机制18k主版
- [[entities/claude-code-skills-mcp-rules-source-analysis|Claude Code Skills / MCP / Rules 源码分析]] — 三个注入位置
- [[entities/claude-code-large-codebase-harness-configuration|Claude Code 在大型代码库中的实战经验：从哪里入手？怎么做对？]] — 大型代码库17k原版
- [[entities/claude-managed-agents-self-hosted-sandbox-mcp-tunnels-enterprise|Claude Managed Agents 新更新\]] — brain/hands分离全版
- [[entities/dingtalk-stream-cli-dual-engine-ai-assistant|钉钉 Stream + CLI 代理双引擎 AI 助手架构]] — Stream+CLI主版
- [[entities/tencent-skill-writing-complete-playbook-jackjchou|鹅厂 Skill 写作完整 Playbook：14 章节 end-to-end 实战 + 工程化评估（腾讯一线踩坑 + Anthropic 官方做法整合）]] — 14章节skill写作playbook
- [[entities/anthropic-mcp-revisited-tool-search-code-orchestration|Anthropic 最新博客：MCP 没死，它又来了]] — MCP三条路
- [[entities/knowledge-work-plugins-shuge-anthropic-deep-source|knowledge-work-plugins拆解：Anthropic官方开源，4 种组件、3 级加载、2 层记忆，纯文件的 AI岗位插件集]] — 岗位级封装+三级披露+两层记忆7956字
- [[entities/anthropic-12-mcp-production-patterns|Anthropic 官方生产级 Agent 最佳实践：12 个可复用的 MCP 设计模式]] — 12个MCP模式
- [[entities/anthropic-managed-agents-scaling|Anthropic Managed Agents：用 K8s 思路虚拟化 Agent 组件]] — 宠物到牛群
- [[entities/claude-code-seven-customization-methods-anthropic-official|Claude Code 七种自定义方法：官方全景指南]] — 七种自定义对比
- [[entities/iqsixinp9lxnkg7avfhfcq|Boris Cherny 新访谈：开发工具正在从 IDE 变成 Agent 控制台]] — Boris新访谈：IDE→Agent控制台控制点迁移
- [[entities/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09|第 09 篇 · Agent 配置：模型、工具、技能、MCP 与提示词的组合]] — 配置驱动架构教程
- [[entities/claude-code-27-tips-engineering-upgrade-jiagoux-2026|Claude Code 27 条技巧：从工具清单到工程升级路径]] — 27技巧全版

## 延伸导航
- [[moc/claude-code-complete-guide|Claude Code 生态完全指南]]
- [[moc/agent-engineering-guide|Agent 工程全景指南]]
- [[moc/rag-knowledge-retrieval|RAG 在知识密集型 Agent 中的最优实践是什么？]]

## 关联

- 同题异语种孪生页：[[entities/anthropic-官方-agent-harness-平台claude-managed-agents-完整指南]]（归并候选，提案卡 #11 批1）
- 同题异语种孪生页：[[entities/anthropic-官方技能最佳实践14-个可复用的-agent-skills-设计模式]]（归并候选，提案卡 #11 批1）

