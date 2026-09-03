---
title: "多模态智能体框架综述：感知融合策略×四模块×四赛道"
created: "2026-08-31"
updated: 2026-08-31
type: entity
tags: [agent, multimodal, perception, fusion, robotics, gui, survey]
sources:
  - raw/articles/multimodal-agentic-frameworks-survey-arxiv-2608-20379
  - raw/articles/multimodal-agentic-frameworks-survey-mozhi-space-2026
confidence: 0.8
provenance_state: extracted
---

# 多模态智能体框架综述：感知融合策略×四模块×四赛道

Maryland/KAUST/Oxford/NUS 等二十余位研究者的60页综述，以三种感知融合策略为主线，分析多模态对智能体感知、推理、记忆、行动四个模块的影响，覆盖四大应用赛道，剖析性能-效率-可扩展性权衡。^[raw/articles/multimodal-agentic-frameworks-survey-arxiv-2608-20379.md]

## 三种感知融合策略

**委托感知（Delegated）**：AI调用外部工具（CLIP等），灵活但信息丢失严重。代表：VisProg、HuggingGPT。^[raw/articles/multimodal-agentic-frameworks-survey-arxiv-2608-20379.md]

**后期融合（Late-fusion）**：ViT编码器+投影层映射到LLM嵌入空间。代表：Flamingo、RT-2。比纯文本好但视觉和语言在高层才融合。^[raw/articles/multimodal-agentic-frameworks-survey-arxiv-2608-20379.md]

**早期融合（Early-fusion）**：所有模态Token化统一处理，最前沿方向。代表：GPT-4o、Gemini。^[raw/articles/multimodal-agentic-frameworks-survey-arxiv-2608-20379.md]

## 关键结论：架构融合比参数量更重要

7B OpenVLA 依靠统一感知-行动token架构，性能超过55B RT-2-X。^[raw/articles/multimodal-agentic-frameworks-survey-arxiv-2608-20379.md]

## 四大应用赛道

### 机器人与具身智能
三阶段演化：SayCan（LLM规划+视觉评估）→ PaLM-E（后期融合）→ VLA端到端（RT-2、OpenVLA）。痛点：云端API延迟、真实环境泛化。^[raw/articles/multimodal-agentic-frameworks-survey-arxiv-2608-20379.md]

### GUI与网页导航
从HTML/XML文本→CogAgent高分辨率微调→GPT-4o原生多模态+SoM标记。WebArena基准SOTA距人类仍60-70%差距。^[raw/articles/multimodal-agentic-frameworks-survey-arxiv-2608-20379.md]

### 多媒体内容生成
从VISPROG/AudioGPT编排工具→GenArtist原生多模态自我纠错。^[raw/articles/multimodal-agentic-frameworks-survey-arxiv-2608-20379.md]

### 长视频理解
VideoAgent选择性检索（20倍效率提升）、VideoMind Chain-of-LoRA。Gemini 1.5百万上下文仍推理薄弱——扩大窗口无法解决推理瓶颈。^[raw/articles/multimodal-agentic-frameworks-survey-arxiv-2608-20379.md]

## 性能-效率权衡

- 融合越深效果越好，但原生多模态API成本延迟高
- 领域微调模型在速度/成本/精度间取得更好平衡
- **面向真实落地，训练领域专用模型长期比持续调用商用API更可行**^[raw/articles/multimodal-agentic-frameworks-survey-arxiv-2608-20379.md]

## 六大局限

1. 接地鸿沟（GUI像素级定位差距巨大）
2. 性能与效率矛盾
3. 长周期记忆脆弱
4. 评测隐患（闭源API无法复现、数据泄露）
5. 对抗鲁棒性不足
6. 多模态幻觉
