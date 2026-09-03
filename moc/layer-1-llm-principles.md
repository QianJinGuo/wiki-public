---
title: 第 1 层全库索引：LLM 技术原理
created: 2026-06-24
updated: 2026-06-24
type: moc
tags: [learning-path, layer-1, llm, principles]
layer: 1
---

# 第 1 层：LLM 技术原理 — 全库索引

> 返回 [学习路径总入口](learning-path.md)

---

## 本层导读

第 0 层给了你「为什么学」，第 1 层给你「它怎么工作」。

你将理解 LLM 的底层机制：Transformer、Token 与上下文窗口、训练三阶段、Scaling Law、推理优化。这层是技术地基——后面所有层都建立在你理解 LLM 机制之上。

---

## 学习路径

```
chap-4 Transformer（50min）→ chap-5 Token 与上下文（50min）→ chap-6 训练三阶段（75min）→ chap-7 Scaling Law（25min）→ chap-8 推理优化（50min）→ 🚪 关卡
```

---

## 本层 concepts

### 架构与注意力
- [[concepts/transformer-architecture|Transformer Architecture]]
- [[concepts/attention-mechanism|Attention Mechanism]]
- Reasoning Models

### Token 与上下文
- [[concepts/llm-tokenizer|LLM Tokenizer]]
- 长上下文技术

### 训练
- [[concepts/llm-pretraining-vs-sft|LLM Pretraining vs SFT]]
- RLHF/DPO/GRPO 对齐
- [[concepts/catastrophic-forgetting|灾难性遗忘]]
- Fine-tuning 技术
- 合成数据生成

### 扩展与优化
- [[concepts/scaling-laws|Scaling Laws]]
- [[concepts/inference-optimization|推理系统优化]]
- [[concepts/model-distillation-compression|模型蒸馏与压缩]]
- GPU 优化

### 可解释性
- [[concepts/mechanistic-interpretability|机制可解释性]]

---

## 本层 entities（精选）

### 架构演进
- [[entities/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl|2026 LLM RL 方法总结]]
- [[entities/kimi-attention-residuals-prenorm-dilution-block-attnres|Kimi AttnRes]]
- [[entities/accelerate-llm-model-loading-and-increase-context-windows-wi|加速 LLM 加载]]

### 训练实践
- [[entities/llm-post-training-full-guide|LLM Post-Training 全景指南]]
- [[entities/aws-grpo-rlvr-sagemaker-math-reasoning|GRPO+RLVR]]
- [[entities/baidu-wenxin-post-training-evolution|百度文心后训练]]

### 推理工程
- [[entities/vllm-v0-to-v1-correctness-before-corrections|vLLM V0→V1]]
- [[entities/sglang|SGLang]]
- [[entities/lightseek-tokenspeed|LightSeek TokenSpeed]]

### 蒸馏与压缩
- [[entities/interconnects-the-distillation-panic|蒸馏焦虑]]
- [[entities/bonsai-image-4b-1-bit-ternary|1-bit 三值量化]]

---

## 本层 raw

- [[raw/articles/kimi-attention-residuals-preNorm-dilution-block-attnres|Kimi AttnRes 论文]]
- [[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws|AWS 基础模型训练]]
- [[raw/articles/vllm-v0-to-v1-correctness-before-corrections|vLLM 论文]]
- [[raw/articles/glm5-scaling-pain-inference|GLM5 推理痛点]]

---

## 🚪 关卡

1. **场景题**：向产品经理解释「为什么 LLM 不能无限记住对话」（Token + 上下文窗口 + KV-Cache）。
2. **费曼题**：向 12 岁孩子解释「attention 在做什么」。
3. **关联题**：Scaling Law 的「Chinchilla 最优」和「灾难性遗忘」有什么看似矛盾实则互补的关系？
4. **场景题**：团队要部署 LLM 但显存不够，从本层学到的 3 个优化方向是什么？
5. **关联题**：回顾第 0 层 Software 2.0。本层讲的训练三阶段，分别对应 2.0 的哪个环节？

---

## 学完这层你应该能

- [ ] 用一张图画出 Transformer 的数据流
- [ ] 解释 token、embedding、attention、context window 四者关系
- [ ] 说出预训练/SFT/RLHF 各自优化什么
- [ ] 解释为什么模型更大效果更好，以及边界
- [ ] 列出 3 种推理优化手段

---

**下一层**：[第 2 层：交互实践](layer-2-interaction.md)
