---

title: "MEC²-TT: 多模态共情对话中的情绪一致性校正与轨迹追踪"
created: 2026-09-02
updated: 2026-09-02
type: entity
tags: [multimodal, empathy, dialogue, emotion, acm-mm, qwen, chain-of-thought]
sources: [raw/articles/acm-mm-2026-mec2tt-multimodal-empathetic-dialogue-ustc]
confidence: 0.75
---

# MEC²-TT: 多模态共情对话中的情绪一致性校正与轨迹追踪

## 核心贡献

中科大陈恩红教授团队提出的 MEC²-TT 是一个多模态共情回复生成框架，被 ACM MM 2026 接收。核心解决两个问题：跨模态情绪信号冲突 + 多轮对话中情绪的持续、积累与转折未被充分建模。 ^[raw/articles/acm-mm-2026-mec2tt-multimodal-empathetic-dialogue-ustc.md]

## 三模块架构

| 模块 | 功能 | 技术方法 |
|------|------|----------|
| **ECC** (Emotion Consistency Correction) | 跨模态情绪一致性校正 | 将文本/语音/视觉映射为情绪概率分布，KLDivergence 衡量模态间差异，自适应校正冲突信号 |
| **ETT** (Emotion Trajectory Tracking) | 历史情绪轨迹建模 | 时间位置编码 + Multi-Head Self-Attention 建模历史情绪依赖；Cross-Attention 以当前轮为 Query 从历史轨迹选择相关信息 |
| **ECoT** (Empathetic Chain-of-Thought) | 结构化共情推理 | Event Scenario → Speaker's Emotion → Emotion Cause → Goal to Response → Strategy to Response 五步推理链 |

## 基座与适配

- 基座模型：Qwen2.5-7B
- 适配方式：轻量级 Linear Adapter 将 ETT 输出映射至 Qwen2.5-7B 文本嵌入空间
- 推理形式：连续 Soft Prompt 参与 ECoT 推理和回复生成

## 实验结果

在两个多模态对话数据集上的表现：

**AvaMERG**（33,048 对话 / 152,021 utterance）：
- BLEU-1: 20.97（消融: ECC-18.37, ETT-18.34, ECoT-9.86）
- Empathy: 3.88（最佳）
- Relevance: 4.07, Fluency: 4.29 ^[raw/articles/acm-mm-2026-mec2tt-multimodal-empathetic-dialogue-ustc.md]

**MELD**（1,400+ 对话 / 13,000 utterance）：
- Empathy: 3.04, Relevance: 3.73, Fluency: 4.61（三项均最佳）
- BERTScore-F1 取得最佳 ^[raw/articles/acm-mm-2026-mec2tt-multimodal-empathetic-dialogue-ustc.md]

消融实验表明 ECoT 贡献最大（BLEU-1 从 20.97 骤降至 9.86），说明结构化推理是连接情绪理解与回复生成的关键环节。 ^[raw/articles/acm-mm-2026-mec2tt-multimodal-empathetic-dialogue-ustc.md]

## 案例分析亮点

1. **表面文本积极但实际负面**：用户说"我没事"但声音颤抖、勉强微笑——基线模型被文本误导给鼓励，MEC²-TT 识别深层愤怒与挫败
2. **话题转移掩盖悲伤**：用户突然从宠物病情转聊"烤蛋糕"——MEC²-TT 将话题转移理解为应对方式，聚焦未缓解的悲伤 ^[raw/articles/acm-mm-2026-mec2tt-multimodal-empathetic-dialogue-ustc.md]

## 论文信息

- 标题：MEC²-TT: Multimodal Emotion Consistency Correction and Trajectory Tracking for Empathetic Dialogue Generation
- 作者：Sirui Zhao, Yu Bai, Jinyang Huang, Fangyuan Liu, Feng-Qi Cui, Xinqi Chen, Guo Cheng, Tong Xu, Enhong Chen
- 单位：中国科学技术大学、合肥工业大学、北京工业大学
- DOI: https://doi.org/10.1145/3767308.3835659

→ [[raw/articles/acm-mm-2026-mec2tt-multimodal-empathetic-dialogue-ustc|原文存档]] ^[raw/articles/acm-mm-2026-mec2tt-multimodal-empathetic-dialogue-ustc.md]