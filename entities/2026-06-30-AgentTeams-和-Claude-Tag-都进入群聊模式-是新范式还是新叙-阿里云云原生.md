---
title: "AgentTeams 和 Claude Tag 都进入群聊模式 是新范式还是新叙 阿里云云原生"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-30-AgentTeams-和-Claude-Tag-都进入群聊模式-是新范式还是新叙-阿里云云原生]
provenance_state: extracted
---

> -> [[raw/articles/2026-06-30-AgentTeams-和-Claude-Tag-都进入群聊模式-是新范式还是新叙-阿里云云原生.md|原文存档]]

sha256: df8dc25a1c589310323343b4161611476e91bcebf87b29bb1db28f131b4da14b ^[raw/articles/2026-06-30-AgentTeams-和-Claude-Tag-都进入群聊模式-是新范式还是新叙-阿里云云原生.md]

## 摘要

文章对比了 Anthropic 的 Claude Tag（把 Claude 作为团队成员嵌入 Slack 频道，ambient 模式下无需被 @ 即可主动监听上下文、依托 Opus 4.8 跨小时异步协作）与阿里云在 520 云峰会发布的 AgentTeams（企业级多智能体治理与协作平台，把群聊抽象为声明式 CRD），讨论 Agent 群聊是否为新范式 ^[raw/articles/2026-06-30-AgentTeams-和-Claude-Tag-都进入群聊模式-是新范式还是新叙-阿里云云原生.md]。文章给出的群聊公式是"多个人类成员 × 多个 Agent × 共享上下文 × 异步任务 × 显式身份权重的协作平面"，并指出需要引入群聊的三类场景：跨领域长链路工作流（注意力在空间与时间上的双重挑战）、多智能体治理（信任边界要求各自归属、授权、计费）、沉淀组织级知识（新成员入群 @ 即可 onboarding） ^[raw/articles/2026-06-30-AgentTeams-和-Claude-Tag-都进入群聊模式-是新范式还是新叙-阿里云云原生.md]。基础设施层面，文章详述了身份权限（K8s ServiceAccount 式的频道身份主体）、凭据治理（所有 Key 托管 Higress AI Gateway、Worker 只持可撤销 Consumer Token、主 Key 二次签发派生凭证、MCP 凭据用完即焚）和三层群体记忆（session 流水、digest 长期记忆、每晚 auto_dream 蒸馏） ^[raw/articles/2026-06-30-AgentTeams-和-Claude-Tag-都进入群聊模式-是新范式还是新叙-阿里云云原生.md]。作者的最终判断：群聊不会取代单聊，它是 AI Native 组织深挖协作效率的试验田——企业 70% 以上协作发生在 IM 群聊，但群聊冷启动代价不低，需要强大模型加上支持其稳定、安全、经济运行的新基础设施 ^[raw/articles/2026-06-30-AgentTeams-和-Claude-Tag-都进入群聊模式-是新范式还是新叙-阿里云云原生.md]。

## 关键要点

- Claude Tag 定义群聊四特征：multiplayer（同一实例与所有人协同）、learns over time（持续积累频道上下文）、takes initiative（ambient 模式主动跟进）、works asynchronously（跨小时跨天任务自主规划）；Anthropic 已用它在产品团队跑通 65% 的 PR
- AgentTeams 角色模型：Manager（人类管理员）、Team Leader（Agent，管理 N 个 Worker）、Worker（最小执行单元），人类分 L1 Admin / L2 Team Leader / L3 Worker 三级权限；每个 Worker 携带 SOUL.md、AGENT.md、MEMORY.md、USER.md 四份声明文件，通信走 Matrix 协议
- AgentLoop 实例：TeamLeader 负责调度与状态，5 个 Worker 分别纳管需求分类、Coding、Test、Review、Verify；需求按复杂度分 T1-T5 五档
- 上下文消歧策略对比：Claude Tag 不主动猜、最大化利用 Slack thread 数据结构；AgentTeams 把消歧责任拆给 TeamLeader 先内部识别分解再下发
- 群体记忆三层：短期记忆落 session/dialog/ 与每日事实卡片；长期记忆走 digest/{personal, procedure, wiki} 目录（本地 Markdown + BM25/Embedding 索引，企业版接 AnalyticDB for PostgreSQL）；auto_dream cron 每晚备份、蒸馏、去重并输出"苏醒汇报"；开源实现为 hiclaw（agentscope-ai/hiclaw），与 QwenPaw 同属 AgentScope，采用 ReMe 记忆框架

## 来源

- 原文：[[raw/articles/2026-06-30-AgentTeams-和-Claude-Tag-都进入群聊模式-是新范式还是新叙-阿里云云原生.md|AgentTeams 和 Claude Tag 都进入群聊模式 是新范式还是新叙 阿里云云原生]]
