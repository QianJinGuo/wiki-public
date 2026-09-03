---
title: "Google Antigravity agents get full context with GitLab Orbit"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/google-antigravity-agents-get-full-context-with-gitlab-orbit]
provenance_state: extracted
---

> -> [[raw/articles/google-antigravity-agents-get-full-context-with-gitlab-orbit.md|原文存档]]

sha256: 1f487eb918dcca1f8afa2f5a3166d7841f5908db7a240c50dd21407711c168db ^[raw/articles/google-antigravity-agents-get-full-context-with-gitlab-orbit.md]

## 摘要

GitLab 官方博客宣布其生命周期上下文图谱产品 GitLab Orbit 上架 Google Antigravity 的 MCP Store，让 Antigravity 中的 Agent 能以结构化方式访问 GitLab 实例中的项目、流水线、merge request、漏洞与源代码。Orbit 将 GitLab 实例索引为知识图谱（组、项目、用户、work item、MR、pipeline、漏洞、代码之间的关系），通过两个 MCP 工具暴露：query_graph（执行结构化查询）与 get_graph_schema（返回可用节点类型与关系）。GitLab 早期内部测试显示，接入了 Orbit 上下文的 Agent 响应最快提升 11 倍、token 消耗减少最多 4.5 倍、幻觉减少最多 45 倍。文章给出三个典型场景（重构影响面分析、新人 onboarding Walkthrough Artifact、结合 Nano Banana Pro 生成实时架构图），并说明 Orbit 与 GitLab Duo Agent Platform 使用同一引擎，支持 Ruby、Java、Kotlin、Python、TypeScript、JavaScript、Rust、C# 八种语言的默认分支索引，变更后数分钟内重新索引；该功能面向 GitLab Premium 与 Ultimate 版本，query_graph 消耗 GitLab Credits，get_graph_schema 免费。^[raw/articles/google-antigravity-agents-get-full-context-with-gitlab-orbit.md]

## 关键要点

- 没有 Orbit 时 Antigravity Agent 只能看到文件和终端，无法知道哪些服务依赖被改代码、历史漏洞、该由谁 review——这些上下文此前靠自定义脚本或复制粘贴
- 两个 MCP 工具：query_graph 执行结构化查询（Agent 用 Orbit 的 JSON DSL 组装查询、返回类型化结果），get_graph_schema 返回节点类型/属性/关系
- 早期内部测试数据：响应最快快 11 倍、token 少用最多 4.5 倍、幻觉最多减少 45 倍
- 三个关键用户旅程：blast radius 分析（一次查询返回依赖方、进行中的 MR 及其 owner）、onboarding Walkthrough Artifact（数分钟内重索引、取代过时 wiki）、依赖架构图（图数据来自实时图谱，用 Nano Banana Pro 渲染，按用户权限过滤）
- Orbit 与 Duo Agent Platform 是同一引擎，无需单独维护上下文管道；安装走 Antigravity MCP Store 的 "Add MCP"，无需配置文件
- 可用性：GitLab Premium/Ultimate on GitLab.com；query_graph 消耗 GitLab Credits，get_graph_schema 免费

## 来源

- 原文: [[raw/articles/google-antigravity-agents-get-full-context-with-gitlab-orbit.md|Google Antigravity agents get full context with GitLab Orbit]]
