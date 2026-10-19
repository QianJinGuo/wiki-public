---
title: "Reflection Beam 1.0：501B 开放权重 MoE 模型与自带 Harness（HF Blog）"
created: 2026-10-19
updated: 2026-10-19
type: entity
tags: [model-release, moe, open-weight, harness, post-training, coding-agent]
sources: [raw/articles/reflection-beam-501b-open-model-hf-blog-2026]
confidence: 0.7
provenance_state: extracted
---

# Reflection Beam 1.0：501B 开放权重 MoE 模型与自带 Harness（HF Blog）

Reflection（一家新的美国实验室）发布首个模型 Beam 1.0，Hugging Face 同日给出 transformers / SGLang / vLLM day-0 支持，并配套推理、SFT 与强化学习配方。Beam 定位在 coding 与 agentic 场景，面向希望在自有硬件、基础设施或云上运行并定制前沿规模模型的团队。^[raw/articles/reflection-beam-501b-open-model-hf-blog-2026.md]

## 架构与能力配置

Beam 是 501B 参数的 MoE LLM，每 token 激活 23B；52 层，每 MoE 层 256 个专家、每 token 路由 8 个。文本进、文本出，上下文窗口 1M token，Apache 2.0 许可。发布方同时给出 BF16、FP8、NVFP4 三种 checkpoint 格式，使同一个模型可以在不同显存与精度预算下部署。^[raw/articles/reflection-beam-501b-open-model-hf-blog-2026.md]

推理侧特性值得注意：模型支持跨轮保留推理内容（`preserve_thinking`）与带专用 grammar 的工具调用；提供 low / medium（默认）/ high / xhigh / max 五档推理强度；tokenizer 内置响应模板；另外附带 DFlash2 推测解码 drafter 模型。这些配置把「推理强度」与「是否保留思考」变成部署时可调的接口，而不是写死在 prompt 里。^[raw/articles/reflection-beam-501b-open-model-hf-blog-2026.md]

## 自带的 Harness

Beam「随模型附带一套 Rust 编写的开源 harness」，官方称其能为该模型提供最佳性能。这一点值得单独记录：模型发布方把 harness（而非仅权重）作为交付物的一部分，说明 agentic 场景的可用性能已经同时取决于模型与执行框架的匹配程度。^[raw/articles/reflection-beam-501b-open-model-hf-blog-2026.md]

## 基准结果

HF Blog 转载了 Reflection 提供的基准表（BF16 / FP8 / NVFP4 三档精度基本持平）：SWEBench Verified 81.6% / 80.2% / 80.7%，SWE Bench Pro v2-Hard 76.5% / 78.4% / 78.6%，Terminal Bench v2.1 80.1% / 78.9% / 78.2%，SWEBench Multilingual 78.3%，SWE Atlas Codebase QnA 31.8%，DeepSWE v1.1 43.3%。推理类：AIME 2026 97.7%，GPQA Diamond 90.6%，LiveCodeBench 88.3%，HLE 无工具 36.0% / 有工具 51.1%，SciCode 49.2%。工具调用与检索类：MCP Atlas 80.8%，BrowseComp（含 context management）76.8%，DeepSearchQA（含 context management）80.5%，Tau3 Banking 36.1%。^[raw/articles/reflection-beam-501b-open-model-hf-blog-2026.md]

注意这些数字由厂商提供，属于自报结果而非独立复现；NVFP4 与 BF16 的差距在多数项目上很小，是「量化后仍可用」的一个参考点。^[raw/articles/reflection-beam-501b-open-model-hf-blog-2026.md]

## 后训练与工程路径

HF 博客给出了完整的后训练配方：用 TRL 做 SFT、用 Miles 做 GRPO（支持全参数 RL）、以及 LoRA 训练路径；推理侧覆盖 transformers、vLLM、SGLang 与 Inference Providers / Endpoints。文章还附了硬件需求小节与三个 demo（在本地 macOS 上构建原生应用、多 agent 扩展、为某个代码库微调决策模型），并从可运行性角度验证「自托管 + 定制」这条路径。^[raw/articles/reflection-beam-501b-open-model-hf-blog-2026.md]

对 wiki 关注面而言，Beam 的价值不在单个 benchmark 数字，而在于它同时覆盖了三条线：开放权重的 frontier-scale MoE 架构（256 专家 / 8 路由 / 1M 上下文）、可复用的 SFT-RL-LoRA 后训练配方、以及模型自带的 Rust harness——三者叠加构成「自托管 agentic 栈」的一种完整形态。^[raw/articles/reflection-beam-501b-open-model-hf-blog-2026.md]

## 关联

- 概念：[[concepts/moe-mixture-of-experts-2025|混合专家模型]]、[[concepts/agent-harness-engineering-paradigm|Agent Harness 工程范式]]、[[concepts/grpo-policy-optimization-2026|GRPO 策略优化]]
- 相关实体：[[entities/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026|异步 GRPO + LoRA 解耦训练]]、[[entities/glm-53-post-training-technical-blog-zhipu-xhs-2026|GLM-5.3 后训练技术博客]]、[[entities/sglang|SGLang]]、[[entities/minimax-h3-h200-sglang-benchmark-2026|MiniMax H3 SGLang 基准]]

→ [[raw/articles/reflection-beam-501b-open-model-hf-blog-2026|原文存档]]
