---
title: 知识库总导航（Topic Map）
created: 2026-04-30
updated: 2026-11-25
type: moc
tags: [moc, navigation, meta, learning-path]
sources: []
confidence: high
---

# 知识库总导航

> 当前快照：9,596 个活跃页面（4,495 entities、178 concepts、36 comparisons、55 queries、45 MOC、38 drafts、4,749 raw sources）。本页只回答“先从哪里进”，完整目录仍以 `index.md` 和主题 MOC 为准。

## 30 秒入口

- 第一次进入：先读 [[moc/wiki-panorama-navigation|知识库全景导航]]，了解规模、分层和当前维护债务。
- 想系统学习：从 [[moc/learning-path|AI 知识体系学习路径]] 开始，再按 `layer-0` 到 `layer-5` 递进。
- 想按主题查：直接跳到下方“主题域入口”。
- 想查维护状态：看 [[queries/wiki-quality-dashboard|Wiki Quality Dashboard]] 与 [[queries/review-queue|Review Queue]]。
- 想参与补页：看 [[moc/wiki-pending-concepts-roadmap|概念补建路线图]] 和 [[queries/negative-results-registry|负结果登记簿]]。

## 推荐阅读顺序

1. [[moc/wiki-panorama-navigation|全景与健康]]：先知道库里有什么、缺什么。
2. [[moc/layer-0-foundation|第 0 层：认知地基]] → [[moc/layer-1-llm-principles|第 1 层：LLM 原理]]。
3. [[moc/layer-2-interaction|第 2 层：交互实践]] → [[moc/layer-3-agent-engineering|第 3 层：Agent 工程]]。
4. [[moc/layer-4-ecosystem|第 4 层：生态工具]] → [[moc/layer-5-production-security|第 5 层：生产与安全]]。
5. 进入具体主题 MOC，最后落到 entity / concept / comparison 和 raw source。

## 主题域入口

### Agent 工程

- [[moc/agent-engineering-guide|Agent 工程全景指南]] — harness、loop、memory、sub-agent、自改进。
- [[moc/loop-engineering|Loop Engineering]] — 从单次任务到持续运行的控制系统。
- [[moc/memory-context-systems|记忆与上下文系统]] — working set、长期记忆、压缩与治理。
- [[moc/multi-agent-coordination|多 Agent 协作]] — 通信、分工、隔离与冲突消解。
- [[moc/workflow-orchestration|工作流编排]] — DAG、状态机、事件驱动与自治边界。
- [[moc/prompt-engineering-guide|Prompt Engineering]] — 提示设计与上下文组织。
- [[moc/ai-skill-design|Skill 设计]] — 可复用能力单元、评测与演进。

### Coding Agent 与运行时

- [[moc/coding-agent-practice|编程 Agent 实践]] — 从 Copilot 到自主编程。
- [[moc/claude-code-complete-guide|Claude Code 生态完全指南]] — 源码、Hooks、Skills、MCP、Managed Agents。
- [[moc/openclaw-architecture|OpenClaw 架构]] — 工具总线、多租户与安全隔离。
- [[moc/tool-use-mcp-patterns|Tool Use 与 MCP]] — 工具接口、协议与生产局限。
- [[moc/mcp-server-patterns|MCP Server 模式]] — 上下文成本、冷启动、隔离与可观测性。

### 模型、训练与推理

- [[moc/llm-core-technology|LLM 核心技术]] — 架构、预训练、后训练、对齐与推理。
- [[moc/llm-research-frontiers|LLM 研究前沿]] — 架构创新、训练方法、可解释性。
- [[moc/reinforcement-learning-rlhf|强化学习与 RLHF/DPO/GRPO]]。
- [[moc/mlops-training-inference|MLOps]] — 训练、推理、量化与模型运维。
- [[moc/nvidia-gpu-acceleration|NVIDIA 与 GPU 加速]]。

### 云、数据与运维

- [[moc/aws-cloud-ai-infrastructure|AWS 云 AI 基础设施]] — Bedrock、AgentCore、SageMaker。
- [[moc/amazon-aws-ai|AWS AI 企业应用]] — 企业 Agent 的 AWS 落地。
- [[moc/cloud-infrastructure|云基础设施]] — Serverless GPU、弹性、成本与多云。
- [[moc/data-infrastructure|数据基础设施]] — Pipeline、治理、数据库与质量。
- [[moc/observability-monitoring|可观测性与监控]] — trace、指标、审计与运维。

### 评测、安全与隐私

- [[moc/evaluation-and-benchmarks|Evaluation & Benchmarks]] — 评测方法论、验证工具与基准。
- [[moc/evaluation-benchmarks-extended|评测体系扩展]]。
- [[moc/security-landscape|AI 安全态势]] — 供应链、Agent 安全、漏洞与平台风险。
- [[moc/security-privacy-landscape|安全与隐私全景]]。
- [[moc/cybersecurity-privacy|网络安全与隐私挑战]]。

### 生态、多模态与研究方向

- [[moc/anthropic-ecosystem|Anthropic 生态与战略]]。
- [[moc/openai-ecosystem|OpenAI 生态]]。
- [[moc/openai-developer-ecosystem|OpenAI 开发者生态]]。
- [[moc/vision-multimodal|视觉与多模态]]。
- [[moc/rag-knowledge-retrieval|RAG 与知识检索]]。
- [[moc/ai-misc-topics-frontier|AI 前沿杂项]]。

## 维护与治理

- [[queries/wiki-quality-dashboard|质量仪表板]] — 当前 lint、v×c、孤儿页与告警快照。
- [[queries/review-queue|复审队列]] — 需要重写、补来源或复核的页面。
- [[moc/wiki-pending-concepts-roadmap|概念补建路线图]] — 骨架页与缺位主题。
- [[queries/wiki-topic-clusters|主题聚类]] — 标签频次与共现结构。
- [[queries/vault-evolution-dashboard|知识库演化仪表板]] — L1–L5 分层和瓶颈。
- [[moc/wiki-structure-navigation|导航设计说明]] — MOC 的组织原则。

## 维护规则

- 新页面优先挂到已有主题 MOC，而不是再建一个平行清单。
- 只有“顺序本身有价值”时才放 `moc/`；有明确结论的专题放 `queries/` 或 `drafts/`。
- 导航页要标快照日期，数字变化时不追着每个页面改，优先更新质量仪表板和本页。
- 历史页面可以保留推断段落，但新写入内容必须有来源或明确标注为推断。
