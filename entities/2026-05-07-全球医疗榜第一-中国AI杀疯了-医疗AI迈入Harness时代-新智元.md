---
title: "全球医疗榜第一 中国AI杀疯了 医疗AI迈入Harness时代 新智元"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-05-07-全球医疗榜第一-中国AI杀疯了-医疗AI迈入Harness时代-新智元]
provenance_state: extracted
---

> -> [[raw/articles/2026-05-07-全球医疗榜第一-中国AI杀疯了-医疗AI迈入Harness时代-新智元.md|原文存档]]

sha256: cde0144d4471ceb4f75b42b0f43aac477f097308e7ad21eeaf23191ade672ca5 ^[raw/articles/2026-05-07-全球医疗榜第一-中国AI杀疯了-医疗AI迈入Harness时代-新智元.md]

## 摘要

文章梳理了 Harness 概念的走红脉络：2026 年 2 月 HashiCorp 联创 Mitchell Hashimoto 在博客中率先命名，随后 OpenAI 发布"3 名工程师、5 个月、0 手写代码、纯靠 Codex Agent 生成 100 万行生产级代码"的实验报告，Martin Fowler 撰写长文、Anthropic 发布长时运行 Agent 的 Harness 设计指南。在此背景下，智诊科技发布面向医疗健康行业的 Agent OS 平台 WiseClaw 2.0，底层延续 OpenClaw 在连接与调度上的能力，上层将 Harness 理念做成系统默认值——分工可概括为"OpenClaw 让 Agent 接得上、调得动、能执行；Harness 让 Agent 跑得稳、管得住、追得回"。文章认为医疗 AI 的门槛已落在四个地方：长时程（服务以月、年为单位）、可追溯、可执行（接设备接系统接流程）、可治理（权限、脱敏、评测、审批、审计）。^[raw/articles/2026-05-07-全球医疗榜第一-中国AI杀疯了-医疗AI迈入Harness时代-新智元.md]

WiseClaw 的关键设计包括：健康档案驱动（客观信息确定性受控读写、服务信息结构化沉淀）；三层流水线（Triage 分诊识别 → Clinical 临床执行 → Evaluator 校验拦截），关键节点插入人工复核；心跳引擎让系统从会话驱动升级为时间、事件、数据共同驱动的主动服务；全链路可观测形成完整 Trace 支持审计回放。底座是自研千亿级 WiseDiag 医疗多模态大模型，在 MedBench、HealthBench 位居第一，最新全球医学 AI 排行榜 DoctorBench 上 WiseDiag-v2 登顶、超越 Google Gemini 和 OpenAI GPT-5.4。商业化方面，公司已与全国 300+ 三甲医院、500+ 头部医疗健康企业合作，并完成 6500 万元天使轮融资，重点发力体检、健康硬件、慢病营养、家庭医生、保险养老五个院外高频场景。^[raw/articles/2026-05-07-全球医疗榜第一-中国AI杀疯了-医疗AI迈入Harness时代-新智元.md]

## 关键要点

- 医疗场景的任务很少"一步到位"，硬塞给单个 Agent 最容易在关键节点失控，所以拆成 Triage/Clinical/Evaluator 三层链路。
- 心跳引擎让系统主动触发：指标异常提醒、复查临近自动唤醒、慢病指标波动风险提示、健康任务未完成干预触达。
- SKILL 模块像乐高一样拼装，可内置审批、脱敏、证据链、医学红线和风控策略。
- Harness 治理能力用四组关键词概括：敢上线（权限/脱敏/边界/门禁）、能交代（证据链/Trace/回放/审计）、用得久（档案/状态/心跳）、管得住（监控/看板/人机协同）。
- 天使轮由杭州千遇智汇、无锡元启联合领投，资金用于 WiseDiag 模型提升、WiseClaw 生态建设、企业级方案深化及好伴 AI 商业化。

## 来源

- 原文: [[raw/articles/2026-05-07-全球医疗榜第一-中国AI杀疯了-医疗AI迈入Harness时代-新智元.md|全球医疗榜第一 中国AI杀疯了 医疗AI迈入Harness时代 新智元]]
