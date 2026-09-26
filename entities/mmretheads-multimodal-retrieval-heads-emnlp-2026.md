---
title: "MMRetHeads：长上下文视觉语言模型的多模态检索头（EMNLP 2026）"
created: 2026-09-26
updated: 2026-09-26
type: entity
tags: [multimodal, retrieval, interpretability, attention, long-context, emnlp-2026, vision-language]
sources: [raw/articles/mmretheads-multimodal-retrieval-heads-emnlp-2026]
confidence: 0.7
---

# MMRetHeads：长上下文视觉语言模型的多模态检索头（EMNLP 2026）

> **来源**：PaperWeekly 转载作者稿（2026-09-25）。论文《Can Retrieval Heads See Images? Multimodal Retrieval Heads in Long-Context Vision-Language Models》收录于 EMNLP 2026，arXiv 2605.27243。数值为论文自报，未见第三方复现。→ [[raw/articles/mmretheads-multimodal-retrieval-heads-emnlp-2026|原文存档]]

## 问题：从"装得下"到"找得到"

长上下文视觉语言模型（VLM）可以接收财报、合同、PPT、研究报告等上百页图文材料，但接收文档不等于正确使用其中的信息。面对文字、图片、表格、图表混排的长上下文，模型首先要回答：与当前问题真正相关的证据在哪里？论文从模型内部机制切入，关注当证据已出现在输入中时，模型内部是否存在一组注意力头负责将问题指向相关的文本或视觉内容。^[raw/articles/mmretheads-multimodal-retrieval-heads-emnlp-2026.md]

## 识别方法：question-to-evidence attention

作者将识别出的注意力头称为多模态检索头（MMRetHeads）。检测方法受 QRHead 启发：分析每个注意力头的 question-to-evidence attention——问题 token 指向标注证据区域的注意力强度。证据是文字时对应文字 token，位于图片中时对应视觉 token；指向证据的注意力越强，该头的检索得分越高。研究覆盖 Qwen3-VL、Gemma3 等 6 个长上下文 VLM，任务包括文本检索、图像检索、渲染文本检索与相同图像检索，并测试 8K 至 128K 五档上下文长度。分析发现文本与图像检索共享一部分注意力头，但不同上下文长度和证据形式会调用不同的头。^[raw/articles/mmretheads-multimodal-retrieval-heads-emnlp-2026.md]

## 因果验证：屏蔽实验

注意力指向证据只说明相关，为检验模型是否真正依赖这些头，论文屏蔽得分较高的检索头并与屏蔽同数量随机注意力头对比：文字与图片检索任务中，屏蔽检索头后模型在不同上下文长度和证据位置下表现显著下降，随机屏蔽影响则小得多。更真实的长文档问答同样如此——MMLongBench-Doc 从 48.2 跌至 5.7，SlideVQA 从 71.2 跌至 8.9，而随机屏蔽后两项任务仍分别保留 32.2 和 52.6 分。多模态推理任务中还能观察到失效表现：检索头被屏蔽后，模型有时在图表仍存在于输入的情况下回答"信息不足"，或读取错误内容、生成不存在的依据。^[raw/articles/mmretheads-multimodal-retrieval-heads-emnlp-2026.md]

## 应用：免训练多模态文档检索

论文进一步把检索头中的证据信号用于多模态文档检索：给定问题，MMRetHeads 为候选页面或版面区域计算相关性分数并排序。页面级检索寻找包含证据的整页，版面级检索进一步定位文字块、表格、图片或图表，整个过程不需要额外训练检索器。在 MMDocIR 上，Qwen3-VL-8B 页面级 Recall@1 达到 64.7（较最强已报告基线 +7.7pp），版面级 Recall@1 达到 39.0（+6.3pp）。^[raw/articles/mmretheads-multimodal-retrieval-heads-emnlp-2026.md]

## 与 wiki 其他机制研究的关联

- 与 [[concepts/attention-mechanism]] 和 [[concepts/mechanistic-interpretability]] 直接相关：这是检索头（retrieval head）假设在多模态场景的扩展——文本-only 场景的 retrieval head 此前已有研究（QRHead），本文证明视觉证据同样被一组可定位的注意力头承载。
- 与 [[entities/dynhd-diffusion-llm-hallucination-detection-denoising-dynamics-emnlp-2026]] 同属 EMNLP 2026 的"模型内部信号外部化"路线：DynHD 用去噪轨迹做幻觉检测，MMRetHeads 用注意力指向做证据定位与检索，两者都把内部动态转化为可操作信号。
- 对 [[concepts/context-management-agent-systems]] 的启示：Agent 处理长文档（Office AI、文档智能体）时，上下文内证据定位的可靠性依赖模型内部检索机制；屏蔽实验给出的 48.2→5.7 崩溃说明"上下文里有答案"远不等于"模型能取到答案"。

## 相关阅读

- [[concepts/rag-retrieval-augmented-generation]] — 免训练检索器路线与经典 RAG 的对照
- [[concepts/retrieval-augmented-generation-rag]] — 检索基础设施视角
