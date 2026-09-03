---
title: "LLM 核心技术 主题地图 (MOC)"
created: 2026-06-15
updated: 2026-06-15
type: moc
tags: [moc, llm, reasoning, alignment, training, post-training, fine-tuning, sft, dpo, grpo, rlhf, 2026]
sources: []
---

# LLM 核心技术 主题地图 (MOC)

> 模型是 Agent 的天花板，但天花板本身也在以 3 个月一代的速度进化。本 MOC 收口 LLM 核心技术：训练方法、对齐技术、推理模型、上下文管理。

---

## 训练方法与后训练

- [[entities/deepseek-v4-training-methodology|DeepSeek V4 训练方法论 — MoE + 多阶段训练路线
- [[entities/deepseek-v4-training-58-page-paper-deep-dive|DeepSeek V4 训练 58 页论文精读 — 完整技术细节
- [[entities/baidu-wenxin-post-training-evolution|百度文心后训练演进 — 后训练策略迭代
- [[entities/build-llm-from-scratch-7-chapters-zion|从零构建 LLM 七章 — 全栈训练教程
- [[entities/building-blocks-for-foundation-model-training-and-inference-on-aws|AWS 基础模型训练推理构件 — 云端训练基础设施

## 对齐技术：RLHF / DPO / GRPO

- [[entities/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl|2026 LLM RL 算法全景 — DeepLog PPO/DPO/GRPO/MARL 对比
- [[entities/aws-sagemaker-sft-dpo-tool-calling|SageMaker SFT/DPO Tool Calling — AWS 微调工具调用
- [[entities/aws-reinforcement-fine-tuning-llm-as-judge|强化微调 + LLM Judge — AWS 的 RL 微调实践
- [[entities/aws-grpo-rlvr-sagemaker-math-reasoning|GRPO/RLVR 数学推理 — AWS GRPO 实验结果
- RLHF/DPO/GRPO 对齐 — 概念层面的三技术对比

## 推理模型

- [[entities/deepseek-v4-flash-means-llm-steering-is-interesting-again|DeepSeek V4 Flash 与 LLM Steering — 推理能力新方向
- [[entities/anthropic|Anthropic LLM 内省机制 — 模型自我认知
- [[concepts/nvidia-telco-reasoning-models-nemo|推理模型 — 概念页

## 上下文与推理效率

- [[entities/accelerate-llm-model-loading-and-increase-context-windows-wi|加速 LLM 加载与扩展上下文 — 推理效率优化
- [[entities/ai-infra-llm-efficient-inference-vllm|vLLM 高效推理 — 推理服务部署
- [[entities/amazon-nova-lite-fine-tuning-cost-effective-vision-detection-model-tuning-case-and-practice|Nova Lite 微调 — 轻量级模型适配

## 训练技术趋势

2026 年的 LLM 训练出现了几个清晰趋势：

1. **后训练比预训练更重要** — 模型架构趋同后，后训练（RLHF/DPO/GRPO）成为差异化关键
2. **验证驱动训练** — LLM-as-Judge 和 Verifier 框架正在反向影响训练目标设计
3. **MoE 主导效率** — DeepSeek V4 的 MoE 架构证明稀疏激活可以在保持质量的同时大幅降低推理成本
4. **多硬件训练** — Anthropic 同时在 Trainium/TPU/GPU 上训练 Claude，供应链对冲成为战略

---

## 与相邻 MOC 的关系

- [[moc/evaluation-and-benchmarks|Evaluation & Benchmarks]] — 训练后的质量验证
- [[moc/anthropic-ecosystem|Anthropic 生态]] — Claude 模型迭代时间线
- [[moc/openai-ecosystem|OpenAI 生态]] — GPT 模型系列
- [[entities/harness-engineering|Harness Engineering — 模型之外的工程层

---

_本 MOC 于 2026-06-15 创建，覆盖训练/对齐/推理/效率四方向。_

## 待关联概念

- [[concepts/ai-team-knowledge-harness|AI Team 知识沉淀体系]]
- [[concepts/grpo-policy-optimization-2026|GRPO 群组相对策略优化]]
- [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|LLM RL 算法演进图谱：从 PPO 到 DPO 到 GRPO 再到 MARL（2026 综述）]]
- [[concepts/moe-mixture-of-experts-2025|MoE 混合专家架构]]
- OpenAI 模型演进
- [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR 基于可验证奖励的强化学习]]
- [[concepts/reinforcement-fine-tuning-rft|Reinforcement Fine-Tuning (RFT)]]
- [[concepts/source-first-knowledge-compilation|source-first 知识编译]]
- [[concepts/context-window-economics|上下文窗口经济学]]
- [[concepts/speculative-decoding|推测解码加速 LLM 推理]]
- 推理引擎对比
- [[concepts/knowledge-base-output-flywheel|知识库输出飞轮]]
