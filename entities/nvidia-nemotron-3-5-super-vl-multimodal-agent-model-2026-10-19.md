---
title: "NVIDIA Nemotron 3.5 Super VL：120B MoE 多模态 Agent 模型与推理侧三件套"
created: 2026-10-19
updated: 2026-11-08
type: entity
tags: [model-release, multimodal, moe, inference-optimization, agent, nvidia]
sources: [raw/articles/nvidia-nemotron-3-5-super-vl-multimodal-agent-model-2026-10-19]
confidence: 0.65
provenance_state: extracted
---

# NVIDIA Nemotron 3.5 Super VL：120B MoE 多模态 Agent 模型与推理侧三件套

NVIDIA 发布 Nemotron 3.5 Super VL：120B 参数 MoE 多模态模型，每 token 激活 12B，把推理、指令遵循、工具调用与文本/图像/文档/视频理解合并在同一个模型内，官方称单 GPU 上 agentic 任务吞吐可达同尺寸级开源模型的最高 5 倍。^[raw/articles/nvidia-nemotron-3-5-super-vl-multimodal-agent-model-2026-10-19.md]

## 定位：常驻 Agent 的专精基座

文章给出的场景是 always-on agent：一次请求可能包含读文档、跨来源核对、遵循公司政策、连续调用多个工具。它覆盖的 agent 工作流包括代码测试与错误分析、跨网站检索与答案核验、从表格/图表/页面版式抽取信息生成报告、漏洞识别与威胁分级、以及搜索与内容审核/事件分析。在模型系统中，它承担需要推理 + 工具 + 图文理解的多步专业工作，高并发文本任务交给更小的模型、复杂规划编排交给前沿模型。^[raw/articles/nvidia-nemotron-3-5-super-vl-multimodal-agent-model-2026-10-19.md]

## 定制与生态数据

开放权重支持 SFT 与 LoRA 教领域行为、RL 提升 agentic 环境表现；随模型发布一系列 Hugging Face 开放数据集（agentic 工具使用与会话数据、金融推理与工具调用、指令遵循与对话、长上下文、公平性、以及 40 个视觉语言数据集），可补充私有数据用于 SFT/RL，多模态部分从 RL 与环境化评测入手。文中给出的行业案例（厂商自报）：Scale AI 为 Skydance 做供应商信用处理自动化，推理成本将降 40 倍、时延降 5 倍；Ramp 把收据图像转结构化 JSON，预期吞吐提升 50%、时延降 10 倍；AI21 与 Alice 用于威胁情报与漏洞分析，准确率提升 41%。^[raw/articles/nvidia-nemotron-3-5-super-vl-multimodal-agent-model-2026-10-19.md]

## 训练路径

模型从 Nemotron 3 Ultra 蒸馏，蒸馏阶段使用的 SkewReverse loss 与其编码能力共同提升编码准确率；后训练采用 multi-teacher on-policy distillation（MOPD），由文本与视觉方向的专门教师对模型自生成回答打分（覆盖视频推理与跟踪、视觉工具使用、对齐）；视觉训练覆盖图表、表格、OCR、多页文档与视频理解，语言与视觉组件分两阶段 SFT，把视觉证据与推理、对话、安全、长上下文数据及 agent 轨迹结合。持续预训练补入 10B Indic token（来自 12 种语言的 Nemotron-pretraining-Indic-v1 子集与自然 Indic 文本），并开源约 2.8T token、22 种印度语言的完整数据集（与 IIT Madras 及 AI4Bharat 合作）。^[raw/articles/nvidia-nemotron-3-5-super-vl-multimodal-agent-model-2026-10-19.md]

## 推理侧三件套：LiLiCorr / NVFP4-QAD / MTP

**LiLiCorr**（开放投机解码技术）不再是逐 token 独立预测候选，而是对整个 draft 的 token 候选做联合评估，用一个轻量网络单次前向计算兼容性分数，产出更连贯的 draft，只保留低成本的顺序最终选择步骤；在 DGX Spark 上作为 Nemotron 3.5 Super VL 低延迟推理的推荐 drafter，token 生成比无投机解码快最多 2.3 倍。**NVFP4 量化 + 量化感知蒸馏（QAD）**：4-bit 权重与激活（Blackwell），同 checkpoint 在非 Blackwell 平台以 4-bit 权重 + 16-bit 激活运行，敏感组件单独选精度；QAD 让量化模型在训练期学习匹配原 BF16 输出，从而闭合精度差，并支持把 KV cache 也存成 NVFP4 而非 FP8；因为发生在训练期，推理时零额外开销。**多 token 预测（MTP）**为模型内置 drafter，一次前向提出多个 token 供主模型验证，并设有专门的对齐主模型的 MTP-boosting 训练阶段。^[raw/articles/nvidia-nemotron-3-5-super-vl-multimodal-agent-model-2026-10-19.md]

## 前缀缓存与 Mamba 状态复用

Agent 反复回传系统指令、工具定义与会话历史，前缀缓存复用这些共享输入的计算。该模型的 serving 优化同时保留 attention cache 与 Mamba 递归状态；额外的 Mamba checkpoint 减少为投机解码恢复输入所需的重放，内部 checkpoint 进一步压缩。这条工程细节值得单独记录：混合架构（attention + Mamba）的 agent serving 需要同时管理两类状态，前缀缓存不再只是 KV 复用问题。^[raw/articles/nvidia-nemotron-3-5-super-vl-multimodal-agent-model-2026-10-19.md]

## 评测口径与边界

评测覆盖 Hermes、LangChain、OpenClaw、OpenCode、Pi、Codex、Claude 等 harness 提供方的自建评测，官方称在 Hermes Bench 与 LangChain 行为评测上高于 Nemotron 3 Ultra，在 SWE-bench/Terminal-Bench 多组设置上优于 Nemotron 3 Super；图表推理 CharXiv 得分 80.8%（对照 63%），配合 Python 代码解释达 84%；EXO Labs 的独立评测把该模型放在 DGX Spark 上准确率-任务完成时间的 Pareto 前沿；上游数据中 Perplexity 提供数据集、Prime Intellect 提供环境用于构建更好的教师模型。需要注意这些数字多为厂商自报或合作方口径，harness 侧评测由其提供方自建。^[raw/articles/nvidia-nemotron-3-5-super-vl-multimodal-agent-model-2026-10-19.md]

对 wiki 关注面的价值：它是一份把「开放权重多模态 agent 模型 + 后训练数据/流程 + 推理优化三件套（投机解码联合评估 / 训练期量化感知蒸馏 / 内建 MTP）」写在同一篇里的工程文档，可与同族 Nemotron 3.5 内容安全实体、以及本库的推理引擎实体对照阅读。^[raw/articles/nvidia-nemotron-3-5-super-vl-multimodal-agent-model-2026-10-19.md]

## 深度分析

### 训练期与推理期的边界正在消失

三个推理优化中有两个把关键工作放在了训练期：QAD 让量化模型在训练中学习匹配 BF16 原模型输出，精度差在训练里闭合，推理时零额外开销；MTP drafter 通过专门的 boosting 阶段对齐主模型。推理效率正被「烘焙」进权重，而不是部署后调参调出来的。对使用者的直接含义：拿到这类 checkpoint 后不要再自行做事后 PTQ，那可能重新打开训练期已经闭合的精度差。

### Agent 效率是多维量，吞吐只是其中一个因子

文章把任务完成时间拆成吞吐、token 用量、工具调用次数、达到正确结果所需步数四个因子，4x 提速来自吞吐与 token 效率的联合而非单纯刷 TPS。对 agent serving，一个话多、多绕几轮工具调用的模型会吃掉吞吐优势，只用 decode 速度做选型会误导。EXO Labs 的独立评测用「准确率-任务完成时间」Pareto 前沿而非单点分数，是更贴近生产口径的评估形状。

### 会话亲和是混合架构 serving 的一等约束

Agent 会话状态现在同时存放在 attention KV cache 与 Mamba 递归状态两处，投机解码恢复输入还要靠额外 Mamba checkpoint 减少重放——调度层不能随意迁移请求。配套 Dynamo 口径：16 张 B200、每用户 50 TPS 下，KV 感知路由 + 会话亲和 + KV offload 可服务 8 倍用户数、首 token 时延降 85%。这把「会话钉在哪个 worker」从性能优化提升为正确性问题：状态跟着会话走。

### 合并视觉栈：复杂度从推理编排迁移到训练管线

单模型合并文本/图像/文档/视频理解的核心价值不在跑分，而在消灭「文本模型 + VLM」双部署之间的任务上下文传递与延迟叠加；代价是训练管线显著变重——语言与视觉两阶段 SFT、视频推理与视觉工具使用等专门教师打分。CharXiv 80.8%（对照 63%，配 Python 代码解释 84%）说明合并后的模型在图表推理上仍能与专门能力竞争。对系统设计者这是一次交易：把复杂度从推理编排层搬到训练管线层。

## 实践启示

1. **用官方发布的 NVFP4 checkpoint，别自行重新量化。** 带 QAD 的模型被事后 PTQ 会重新打开训练期已闭合的精度差；Blackwell 用 4-bit 权重+激活，非 Blackwell 用 4-bit 权重 + 16-bit 激活，并按组件敏感性保留高精度；KV cache 可试 NVFP4 替代 FP8 省显存，但需复测精度目标。

2. **混合架构 serving 栈必须同时管理两类状态。** 前缀缓存的淘汰与换入策略要分别覆盖 attention KV 与 Mamba 递归状态；保留 Mamba checkpoint 以降低投机解码恢复输入的重放成本，并把重放开销纳入压测指标。

3. **给 agent 负载上会话亲和路由。** 路由器实现 KV 感知路由 + session affinity，把同一会话钉在同一 worker，配合 KV offload；参考量级：16×B200、每用户 50 TPS 下支撑 8 倍用户数、TTFT 降 85%。

4. **评测以任务完成时间与 token 用量为准，不只 decode TPS。** 复现「准确率-完成时间」Pareto 图做选型；对厂商自报倍数（40x 成本、10x 时延、41% 准确率）索要方法论，harness 自建评测只作方向性参考。

5. **后训练之后重新校准 drafter。** 自有数据的 SFT/RL 会破坏 MTP-boosting 建立的 drafter-主模型对齐，需重跑对齐训练或复测投机解码接受率；按平台选 drafter（DGX Spark 推荐 LiLiCorr，最高 2.3x）。

6. **多模态域适配优先 RL + 环境化评测。** 官方对多模态定制的推荐起点是 RL 与环境化评测而非纯 SFT；合并视觉栈部署前先预算训练侧成本（两阶段 SFT、专门教师、视觉语言数据集筛选）。

## 关联

- 概念：[[concepts/moe-mixture-of-experts-2025|混合专家模型]]、[[concepts/inference-optimization|推理优化]]、[[concepts/agent-harness-engineering-paradigm|Agent Harness 工程范式]]
- 相关实体：[[entities/nemotron-3-5-content-safety|Nemotron 3.5 内容安全]]、[[entities/nemotron-3-5-content-safety-multimodal|Nemotron 3.5 多模态内容安全]]、[[entities/nvidia-nemotron-3-ultra-sagemaker-jumpstart-moe-agentic|Nemotron 3 Ultra SageMaker JumpStart]]、[[entities/nvidia-nemotron-3-agents-rag-voice-safety|Nemotron 3 Agents / RAG / Voice / Safety]]、[[entities/meituan-slm-bounded-generation-inference-engine-2026|美团 SLM 推理引擎]]

→ [[raw/articles/nvidia-nemotron-3-5-super-vl-multimodal-agent-model-2026-10-19|原文存档]]
