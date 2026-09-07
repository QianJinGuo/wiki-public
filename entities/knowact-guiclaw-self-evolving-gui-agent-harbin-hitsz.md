---
title: "KnowAct-GUIClaw — 哈工大「Know Deeply, Act Perfectly」自进化 GUI Agent"
created: 2026-08-19
updated: 2026-09-07
type: entity
tags: [gui-agent, openclaw, self-evolution, agent-memory, long-horizon, mobileworld, hitsz, task-orchestration, tool-use]
sources: [raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# KnowAct-GUIClaw — 哈工大「Know Deeply, Act Perfectly」自进化 GUI Agent

## 概述

哈工大张民教授立知大模型团队开源 **KnowAct-GUIClaw**，面向个人助手提出 **「Know Deeply, Act Perfectly」**（知道得越深，行动得越准）范式，在长程 GUI 任务评测基准 MobileWorld 上达到 SOTA。论文 arxiv 2607.12625，代码 github.com/HITsz-TMG/KnowAct。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md]

## 核心问题

仅给 OpenClaw 通用智能体「外挂」一个 GUI Agent，仍会遇到：跨应用信息易丢失、操作依赖不断变化的图形界面、执行轨迹用完即弃。许多现有 GUI 智能体每次面对任务都像第一次使用设备——走过的弯路不会成经验，成功操作无法沉淀为可复用能力。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md]

## 架构：主智能体编排 + 可插拔 GUI 智能体

框架以具备长期上下文、记忆和工具能力的主智能体负责任务编排，以可插拔的 GUI 智能体负责实时界面操作，在二者间实现「知道得深 → 行动得准」的自进化闭环。它直接回答「只用一句自然语言指令，让个人 AI 助手跨 App 连续干活（提取会议地址→转发联系人→打开地图规划路线）」这一场景。^[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz.md]

## 意义

KnowAct-GUIClaw 与 [[entities/mobileforge-annotation-free-gui-agent-kuaishou-zju-2026|MobileForge]]、[[entities/让gui-agent不再边做边忘快手浙大提出memgui-agent攻克长程gui任务|MemGUI]]、[[entities/saas-bench-gui-agent-eval-unipat|SaaS-Bench]] 同属 GUI Agent 前沿，其「主智能体记忆编排 + 自进化」路线与 [[entities/拆解-openclaw-架构一6-阶段流水线与-20-平台的消息归一化|OpenClaw]] 生态深度耦合。

→ [[raw/articles/knowact-guiclaw-self-evolving-gui-agent-harbin-hitsz|原文存档]]
