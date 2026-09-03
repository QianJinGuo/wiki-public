---
title: "AWS AI 服务如何支撑企业级 Agent 应用？"
created: "2026-05-15"
updated: "2026-05-21"
type: moc
tags: [topic-map, navigation, amazon-aws-ai, agent, enterprise, bedrock]
sources:
  - raw/articles/aws-ai-enterprise-agent-overview
---

# AWS AI 服务如何支撑企业级 Agent 应用？

> AWS 通过 Bedrock、SageMaker、QuickSight 等服务为企业级 Agent 应用提供全栈支撑，从模型服务到编排框架形成完整生态

## 核心支撑架构

### 1. Amazon Bedrock — 企业级 Agent 的模型基座

Bedrock 是 AWS 企业级 AI 的核心，提供 Claude、Llama、Titan 等主流模型的托管服务：

- **Halliburton 案例**：基于 Bedrock 的地震工作流实现生成式 AI 驱动的油气勘探分析 ^[raw/articles/aws-bedrock-halliburton-seismic-workflow-genai.md]
- **GenAI 消息防御**：100% 检测混淆联系人信息的安全应用场景 ^[raw/articles/aws-bedrock-intelligence-message-defense.md]
- **Model Agility Framework**：6 步 LLM 跨代际迁移框架，帮助企业平滑升级模型版本 ^[raw/articles/aws-generative-ai-model-agility-framework.md]

### 2. SageMaker — 训练与推理基础设施

SageMaker 提供企业级机器学习基础设施，支持 Agent 的定制化训练：

- **GRPO+RLVR**：Qwen 数学推理 3.7x 提升的工程细节，展示了 RLHF 在 Agent 对齐中的应用 ^[raw/articles/aws-grpo-rlvr-sagemaker-math-reasoning.md]
- **LLM-as-Judge**：RFT 的 6 步法官设计方法论，用于 Agent 输出质量评估 ^[raw/articles/aws-reinforcement-fine-tuning-llm-as-judge.md]
- **容量感知推理**：实例池+优先级 Fallback 机制保障 Agent 服务稳定性 ^[raw/articles/aws-sagemaker-capacity-aware-inference-fallback.md]
- **GPU 虚拟化**：基于 MIG 技术的 GPU 虚拟化最佳实践，优化多租户 Agent 部署成本 ^[raw/articles/gpu-virtualization-using-mig-technology-on-amazon-sagemaker-hyperpod.md]

### 3. Amazon QuickSight — Agent 驱动的数据分析

QuickSight 作为 BI 工具，正在成为 Agent 输出可视化的重要载体：

- **Dataset QA**：NL 直查 S3 Iceberg，实现自然语言驱动的数据分析 ^[raw/articles/aws-quicksight-dataset-qa-natural-language.md]
- **TARA 案例**：准确率+48%、延迟-90% 的威胁评估与风险分析场景 ^[raw/articles/aws-quicksight-dataset-qa-tara-case.md]
- **企业数据到 AI 决策**：加速企业数据到 AI 驱动决策的路径 ^[raw/articles/amazon-quick-accelerating-the-path-from-enterprise-data-to-ai-powered-decisions.md]

## 企业级 Agent 应用场景

### 1. 制造业智能化

- **Amazon Nova Manufacturing Intelligence**：多模态 Embeddings 在制造业的智能应用 ^[raw/articles/amazon-nova-manufacturing-intelligence.md]
- **Fine-Tuning 案例**：高性价比的视觉检测模型微调案例与实践 ^[raw/articles/amazon-nova-lite-fine-tuning-cost-effective-vision-detection-model-tuning-case-and-practice.md]

### 2. 金融与合规

- **EU AI Act 合规**：LLM 微调的 EU AI Act 合规要求指南 ^[raw/articles/navigating-eu-ai-act-requirements-for-llm-fine-tuning-on-amazon-sagemaker-ai.md]
- **SunFinance ID 提取**：Textract+Claude 准确率 90.8% 的 ID 提取方案用于欺诈检测 ^[raw/articles/aws-sun-finance-ai-id-extraction-fraud-detection.md]

### 3. 供应链与物流

- **Amazon Supply Chain Services**：面向各种规模企业的供应链服务 ^[raw/articles/amazon-supply-chain-services.md]
- **Hapag-Lloyd**：1.5 万反馈/月 95% 情感准确率的客户反馈分析 ^[raw/articles/aws-hapag-llody-bedrock-customer-feedback]

## Agent 架构支撑

### 实时语音与多模态

- **Nova Sonic + WebRTC**：基于 Amazon Nova Sonic 和 WebRTC 构建实时语音流应用 ^[raw/articles/build-real-time-voice-streaming-with-amazon-nova-sonic-and-webrtc.md]
- **Foundation Model Training & Inference**：AWS 上基础模型训练和推理的构建模块 ^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]

### 数据管道与集成

- **AIDLC 范式**：平台驱动到大数据工程的范式迁移 ^[raw/articles/aws-aidl-paradigm-shift-platform-driven-data-engineering.md]
- **MLflow v3.10**：生成式 AI 开发新特性，支持 Agent 生命周期管理 ^[raw/articles/aws-mlflow-v310-generative-ai-development.md]
- **跨账户 Athena**：打破数据孤岛，实现跨账户统一洞察 ^[raw/articles/from-siloed-data-to-unified-insights-cross-account-athena-access-for-amazon-quic.md]

### 迁移与集成工具

- **EZConvertBI**：Power BI/Tableau 到 QuickSight 的自动迁移，降低迁移成本 ^[raw/articles/aws-transform-ezconvertbi-bi-migration.md]
- **U.S. Bank 案例**：关键应用迁移到 AWS 以推进 AI 能力 ^[raw/articles/us-bank-aws-ai-migration.md]
- **DynamoDB 增量同步**：基于 Kinesis 的历史数据清理与增量同步方案 ^[raw/articles/基于-amazon-kinesis-data-streams-实现-dynamodb-历史数据清理与增量同步.md]

## 相关实体

- [[entities/amazon-bedrock-model-inference-serverless-architecture-case-study|Amazon Bedrock 无服务器推理架构 — Bedrock 企业级模型服务
- [[entities/amazon-nova-manufacturing-intelligence|Amazon Nova Manufacturing Intelligence — 多模态 Embeddings 制造业应用
- [[entities/build-real-time-voice-streaming-with-amazon-nova-sonic-and-webrtc|Nova Sonic 实时语音流 — 多模态语音生成
- [[entities/gpu-virtualization-using-mig-technology-on-amazon-sagemaker-hyperpod|SageMaker HyperPod MIG GPU 虚拟化 — 多租户 Agent 部署优化
- [[entities/aws-quicksight-dataset-qa-tara-case|QuickSight TARA 案例 — Agent 驱动的威胁评估分析
- [[entities/aws-bedrock-halliburton-seismic-workflow-genai|Bedrock Halliburton 地震工作流 — 生成式 AI 油气勘探

## 相关概念

- [[concepts/harness-engineering-framework|Harness Engineering 框架 — Agent 编排与 Harness 工程
- [[concepts/agent-orchestration-patterns|Agent Orchestration Patterns]] — 多步骤 Agent 任务编排
- [[concepts/inference-optimization|推理系统优化]] — LLM 推理性能与成本优化

## 相关主题

- [[moc/cloud-infrastructure|Cloud Infrastructure]] — 云基础设施架构
- AI Agent Platforms Topic Map（已删除） — Agent 平台对比
- [[queries/ai-agent-era-developer-toolchain-redesign|Developer Tooling]] — 开发者工具链
- [[queries/digital-commerce-ai-agent-scenarios-challenges|Digital Commerce & Web3]] — 电商场景应用

## 待关联概念

- [[concepts/cloud-ai-infrastructure|Cloud AI Infrastructure]]
- NVIDIA GPU 生态
- 医疗 AI 应用
