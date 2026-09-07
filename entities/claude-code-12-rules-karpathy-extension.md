---
type: entity
title: "CLAUDE.md 12 条规则：Karpathy 扩展模板"
created: 2026-05-12
updated: 2026-09-07
sources:
  - [[raw/articles/claude-code-12-rules-karpathy-extension|Claude写代码错误率从41%降到11%]]
tags: [agent-prompting, claude-code, coding-agent, prompt-engineering, best-practice]
author: "Mnimiy (@Mnilax)"
review_value: 9
review_confidence: 8
review_recommendation: strong
review_stars: 5
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: dup-correction: 同主题保留更全版本; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# CLAUDE.md 12 条规则：Karpathy 扩展模板

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/claude-code-12-rules-karpathy-extension.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/hermes-agent-deep-dive|Hermes Agent 深度解析（阿里云/飞樰）]] — 自进化内外双路径+四维工程7934字rv9全版
- [[entities/wangyunhe-harness-optimization-agentsoul|王云鹤眼中的Harness：复杂优化问题，AGI灵魂争夺之战]] — Agent=Models+Harness联合优化
- [[entities/agentmemory-coding-agent-local-memory|AgentMemory：Coding Agent 本地记忆系统]] — 本地记忆运行时
- [[entities/deepseek-v4-ds4c-antirez-local-inference-qbitai|DeepSeek V4 DS4C Antirez 本地推理实践]] — ds4.c非对称量化
- [[entities/claude-code-origin-safety-alignment-boris-2026|Claude Code 身世：从安全对齐到开发工具的革命]] — 安全对齐起源

## 工程实践
- [[entities/karpathy-claude-md-rules|Karpathy CLAUDE.md — 四条行为准则让 AI 编程 Agent 减少结构性失败]] — CLAUDE.md四行为准则rv9
- [[entities/claude-code-skills-practical-guide-discovery-frontmatter|Claude Code Skills 实战指南 — 发现机制、编写与安全]] — 发现机制与安全
- [[entities/claude-md-12-rules-mnilax|CLAUDE.md 规则从 Karpathy 的 4 条增加到 12 条]] — 12规则rv9主版
- [[entities/claude-code-skills-mcp-rules-source-analysis|Claude Code Skills / MCP / Rules 源码分析]] — 三个注入位置
- [[entities/claude-code-prompt-source-analysis|Claude Code Prompt 提示词体系源码解析]] — 六大prompt模块全版
- [[entities/harness-engineered-business-agent-evaluation-aliyun-boyu|Harness 工程搭建式业务 Agent 评测方案：Claude Code 作 Harness 搭建者]] — CC搭评测Harness，1.5周→1-2天
- [[entities/deepseek-code-harness|DeepSeek Code Harness]] — DSH 28k主版
- [[entities/tencent-skill-writing-complete-playbook-jackjchou|鹅厂 Skill 写作完整 Playbook：14 章节 end-to-end 实战 + 工程化评估（腾讯一线踩坑 + Anthropic 官方做法整合）]] — 14章节skill写作playbook
- [[entities/claude-code-skill-writing-guide|Claude Code SKILL.md 写作指南]] — SKILL.md写作
- [[entities/ljg-skills-deep-dive-datastudio-2026|李继刚 23 个 Skills 深度拆解——认知工序流水线]] — 李继刚23 Skills认知工序流水线拆解
- [[entities/claude-code-why-instructions-ignored-jia-gou-x-2026|Claude Code 为什么会忽略指令：四类失效原因 + 五层规则框架]] — 指令失效四类
- [[entities/claude-code-dynamic-workflows-thariq-practical-patterns|Claude Code Dynamic Workflows 实战模式与构建技巧]] — 3失败6模式11用例
- [[entities/claude-code-html-artifacts|Using Claude]] — HTML unreasonable effectiveness主版
- [[entities/300万人在存的claude提示词|300万人在存的Claude提示词]] — 提示词工程框架
- [[entities/harness-engineering-systematic-explainer|Harness Engineering 系统性解读]] — 李宏毅课程解读7933字最全版
- [[entities/agent-browser-zombie-process-cleanup-qoderwork-2026|Agent Browser 僵尸进程排查与定时清理（Claude Code + QoderWork 实战）]] — 僵尸进程排查自愈实践
- [[entities/claude-code-checkup-feature-boris-cherny-2026|Claude Code /checkup 功能：清理 Skills/MCP 提升性能]] — 配置腐烂治理
- [[entities/autoresearch-agent-algorithmic-development-file-compression-2026|Autoresearch: AI Agent-Driven Algorithmic Development (File Compression Experiment)]] — 压缩算法autoresearch实验
- [[entities/thariq-fable-5-usage-mindset-map-territory-unknown-unknowns|Thariq（Claude Code工程师）的Fable 5使用心法：地图≠领土，用未知消除法突破模型瓶颈]] — 地图领土未知消除法
