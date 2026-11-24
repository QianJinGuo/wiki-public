---
title: "Reflection Beam 1.0：501B 开放权重 MoE 模型与自带 Harness（HF Blog）"
created: 2026-10-19
updated: 2026-11-14
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

## 深度分析

### 「自有、定制、可扩展」：开放前沿权重对闭源实验室的定位挤压

Beam 的卖点不在于某个单项 benchmark 刷新纪录，而在于它把「前沿规模 + 开放权重 + 全套配方」打包成了一个完整的生态位命题：HF Blog 的标题就是「a 501B open model you can own, customize, and scale」。对闭源前沿实验室而言，这意味着竞争维度从「API 智能水平」扩展到「谁允许用户拥有并改造智能本身」——开放阵营第一次在前沿规模上同时给出了可自托管（NVFP4 约 280GB、4×B200 可服务）、可微调（LoRA SFT 单节点 8×B300 即可跑）与可强化学习（全参数 GRPO 配方完整公开）的三条路径。这实际上把「数据敏感、需要深度定制、有自有算力」这一类此前只能被动接受闭源 API 条款的高价值用户，变成了开放模型的天然主场。^[raw/articles/reflection-beam-501b-open-model-hf-blog-2026.md]

### Harness 即产品：模型交付物的边界在重新划定

最值得单独记录的信号是「随模型附带一套 Rust 开源 harness」这个交付物形态。传统上模型发布只交付权重 + tokenizer，harness 属于下游框架（SGLang、各 coding agent）的事；Beam 把 harness 纳入官方交付并声称「为该模型提供最佳性能」，等于承认了一个事实：agentic 场景的可用性能 = 模型 × harness 的匹配度，而非模型单独决定。这种「harness-as-product」策略与模型自带的推理强度五档配置（low/medium/high/xhigh/max）、`preserve_thinking` 开关、专用 grammar 工具调用共同构成一个设计取向——把原先散落在 prompt 工程里的 agentic 行为控制，上移为部署接口。如果这个模式被后续开放模型跟随，第三方 harness 生态将被迫从「通用适配」转向「与特定模型联合调优」的分工。^[raw/articles/reflection-beam-501b-open-model-hf-blog-2026.md]

### 规模化 GRPO 后训练：配方公开本身就是基础设施

后训练部分的信息密度极高：全参数 GRPO 在 96×H200（64 训练 + 32 rollout）上跑通，AIME25 从 46.25% 提升到 55.8%；LoRA GRPO 只需 32×H200，反而把 AIME25 从 42.9% 拉到 64.2% 且输出长度从 3188 降到 2197 token——LoRA 在这个配方里不是「降级替代」而是更优的性价比点。配合 Megatron 训练 + SGLang rollout + Miles 权重发布的解耦架构、以及节点本地 NVMe 流式优化器状态等工程细节，这份配方相当于把「前沿规模 RL 后训练」从传闻变成了可照抄的 runbook。对开放生态而言，其意义与权重本身相当：闭源实验室的 RL 训练栈一直是黑箱，而 Beam 的发布让「一个 501B MoE 在真实 checkpoint 上验证过的 GRPO 配置（TP8/EP8/DP8、截断重要性采样、无 KL 惩罚）」成为公共知识。^[raw/articles/reflection-beam-501b-open-model-hf-blog-2026.md]

### 基准定位与部署经济学：自报数字之外的两个硬信号

厂商自报的基准（SWEBench Verified 81.6%、AIME 97.7%、GPQA 90.6%）需要打折看待，但表里有两个不易注水的硬信号：一是 BF16/FP8/NVFP4 三档精度差距普遍在 1-2 个百分点内，NVFP4 把部署成本从 1TB 压到 280GB，这是「4-bit 量化在后训练充分的大模型上基本无损」的又一个实证点；二是混合注意力带来的 KV cache 经济学——43 层滑窗（512 token）+ 9 层全局注意力使 1M 上下文的 cache 从理论 208 GiB 降到 36 GiB，约六分之一。这两点共同回答了「501B 模型到底要多少钱才能跑起来」：四张 B200 即可服务，且长上下文场景的成本不再随 token 数线性爆炸。对部署规划者而言，这组数字比 benchmark 排名更有决策价值。^[raw/articles/reflection-beam-501b-open-model-hf-blog-2026.md]

## 实践启示

1. **评估开放前沿模型时把 harness 纳入测试变量**：Beam 官方明示其 Rust harness 才能拿到最佳性能，这意味着同一权重换 harness 可能得出完全不同的 agentic 成绩；自建评测时至少固定 harness 版本，最好做一次跨 harness 对照。
2. **部署选型优先 NVFP4（Blackwell）/FP8（Hopper 及更早）**：官方硬件小节给出的参考是 NVFP4 约 280GB、4×B200（720GB）留足 KV cache 与并发余量；BF16 1TB checkpoint 主要用于训练与精度对照，不是服务起点。
3. **RL 后训练从 LoRA GRPO 起步而非全参数**：原始数据显示 LoRA GRPO（32×H200）比全参数 GRPO（96×H200）用约三分之一算力取得更高 AIME25 提升（42.9%→64.2%），先验证任务可学习性再考虑全参数投入。
4. **把推理强度档位与 `preserve_thinking` 当作运维参数调优**：HF 的 vibe eval 显示 low 档在长上下文格式遵从与数学论证上会失败而 high 档通过，这类开关的性价比远高于反复改 prompt，应进入部署配置而非 prompt 模板。
5. **长上下文成本核算用混合注意力模型**：1M token 场景每序列 KV cache 约 36 GiB（BF16）而非朴素的 208 GiB，容量规划时按全局层数（9 层）而非总层数（52 层）估算增长。
6. **自报 benchmark 一律要求复现路径**：Beam 的数字由 Reflection 提供、HF 转载，官方同时公开了 vibe eval 的完整 trace bucket（prompt、输出、grader）——把「要求厂商公开评测 trace」作为评估任何新模型的门槛条件。^[raw/articles/reflection-beam-501b-open-model-hf-blog-2026.md]

## 关联

- 概念：[[concepts/moe-mixture-of-experts-2025|混合专家模型]]、[[concepts/agent-harness-engineering-paradigm|Agent Harness 工程范式]]、[[concepts/grpo-policy-optimization-2026|GRPO 策略优化]]
- 相关实体：[[entities/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026|异步 GRPO + LoRA 解耦训练]]、[[entities/glm-53-post-training-technical-blog-zhipu-xhs-2026|GLM-5.3 后训练技术博客]]、[[entities/sglang|SGLang]]、[[entities/minimax-h3-h200-sglang-benchmark-2026|MiniMax H3 SGLang 基准]]

→ [[raw/articles/reflection-beam-501b-open-model-hf-blog-2026|原文存档]]
