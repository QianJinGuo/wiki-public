---
title: "Observability Platform"
created: 2026-07-27
updated: 2026-08-01
type: entity
tags: ["observability", "monitoring", "platform"]
sources: [raw/articles/llm-observability-4-layer-quant67]
provenance_state: extracted
confidence: 0.6
---

# Observability Platform

> -> [[raw/articles/llm-observability-4-layer-quant67.md|原文存档]]

## 概述

可观测性在传统微服务已经是老生常谈：Metrics + Logs + Traces 三件套 + 一点 Profiling 就能覆盖 90% 排障。但直接搬到 LLM 系统上远远不够： - 成本不是"CPU 秒"而是"token × 单价"，input/output/cached 三档价格不同 - 延迟不是单一 latency，要拆 TTFT（Time To First Token）/ TPOT（Time Per Output Token）/ E2E - 一个 Agent 请求可能产生 20 次子 LLM 调用、5 次工具调用、3 次 retriever - 请求返回 HTTP 200、延迟正常、成本正常，**但答案是幻觉** — 传统监控一个告警都不会响 文章目标：把 LLM 可观测性拆成 4 层，串联主流通用栈，目标是"出问题时 5 分钟定位、3 小时修复，下次不再出现"。 ^[raw/articles/llm-observability-4-layer-quant67.md]

## 主要内容

- 文章定位
- 1. 为什么 LLM 需要新的可观测性
- 1.1 与传统微服务差异
- 1.2 四层观测模型
- 2. 核心指标体系
- 2.1 延迟：TTFT / TPOT / E2E
- 2.2 吞吐：tokens / req / GPU
- 2.3 GPU 与引擎内部信号

## 来源

- [[raw/articles/llm-observability-4-layer-quant67.md|原文存档]]
- 原始链接: https://quant67.com/post/llm-infra/23-observability/23-observability.html
