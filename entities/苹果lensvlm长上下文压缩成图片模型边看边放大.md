---
title: "苹果LensVLM：长上下文压缩成图片，模型边看边「放大」"
created: 2026-10-15
updated: 2026-10-20
type: entity
tags: [llm, multimodal, context-management, compression, agent, vision-language]
sources: [raw/articles/苹果lensvlm长上下文压缩成图片模型边看边放大]
confidence: 0.7
provenance_state: extracted
---

# 苹果LensVLM：长上下文压缩成图片，模型边看边「放大」

## 核心机制

苹果基础大模型团队与杜克大学提出的 LensVLM 解决长上下文压缩的单行道问题：多数压缩方案在压缩后就无法回头查阅原文，细节一旦被摘要丢掉就找不回来。LensVLM 把「定位证据」与「读取证据」分成两步——模型先浏览覆盖全文的压缩页面（文字渲染成图像，以更少的视觉 token 承载全文），再针对与问题相关的页面调用 `read_page` 展开工具取回原文并加入上下文；证据不足时可继续选下一页，相当于给模型一个可反复使用的「放大镜」。 ^[raw/articles/苹果lensvlm长上下文压缩成图片模型边看边放大.md]

## 关键数据

- 七项文本问答评测：**4.3× 有效压缩**下平均准确率 **68.9%**，接近完整文本参照的 **72.4%**。
- 基线对比：基础模型直接阅读 5× 压缩图像仅 **31.3%** 准确率——证明纯压缩（无展开机制）会被视觉编码器辨识能力卡死，字符压得越小越难辨认。
- 压缩率可通过字号、页面尺寸、文字密度调节，无需重设文本词表。

## 与 DeepSeek-OCR 路线的对比

DeepSeek-OCR 展示了「图像承载文字」的同一大方向，但压缩页面清晰度是瓶颈——压得太狠答案在页面上模型却看不清。LensVLM 的增量在于承认压缩有损，用 agentic 工具调用（选页 → read_page → 继续选页）补回被压缩掉的细节，形成「压缩保全文覆盖 + 展开保细节精度」的双层结构。 ^[raw/articles/苹果lensvlm长上下文压缩成图片模型边看边放大.md]

## Agent 长程任务意义

Agent 执行长程任务时，代码、工具输出和交互记录不断累积，上下文管理面临「压缩省 token vs 丢失后续关键线索」的两难。LensVLM 的选页-展开循环给出一种可迁移模式：压缩层负责广度（全文都在视野内），工具层负责深度（按需取回原文），与 [[concepts/context-management-agent-systems|Agent 上下文管理]] 中的工作集分层思路互补，也为 [[concepts/context-window-economics|上下文窗口经济学]] 提供了新的压缩率-准确率数据点。该工作已被 NeurIPS 2026 接收。 ^[raw/articles/苹果lensvlm长上下文压缩成图片模型边看边放大.md]

## 资源

- 论文：arxiv.org/abs/2605.07019
- 模型：huggingface.co/apple/LensVLM-9B
- 代码：github.com/apple-aiml-research/ml-lensvlm
- 9B Demo 已上线 Hugging Face Spaces

→ [[raw/articles/苹果lensvlm长上下文压缩成图片模型边看边放大|原文存档]]
