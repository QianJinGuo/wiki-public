---
title: "EMNLP 2026 | 多模态大模型视觉概念的因果探测框架（上交大 × 蚂蚁）"
created: 2026-10-20
updated: 2026-10-20
type: entity
tags: [multimodal, interpretability, activation-steering, vision-language, probing, emnlp]
sources: [raw/articles/emnlp-2026-multimodal-visual-concept-causal-probing-paperweekly]
confidence: 0.8
---

# EMNLP 2026 | 多模态大模型视觉概念的因果探测框架

上海交通大学与蚂蚁集团的研究把「模型内部怎么编码视觉概念」从被动观察推进到主动因果干预：用成对图像提取概念方向向量，再按强度 α 注入指定层，看模型是否被引导输出该概念。论文为《Causal Probing for Internal Visual Representations in Multimodal Large Language Models》（arXiv 2605.05593），配套数据集 Vis_diff 已开源。^[raw/articles/emnlp-2026-multimodal-visual-concept-causal-probing-paperweekly.md]

## 方法：两步因果干预

第一步**提取概念**：准备成对图片（同一场景有猫 / 无猫），对比各层内部激活差异，得到代表该概念的方向向量。第二步**注入干预**：处理新图时把该向量按系数 α 加到指定网络层，观察输出是否被引导到对应概念。实验覆盖 Qwen2.5-VL、Qwen3-VL（8B/32B）、Gemma3（4B/27B）、LLaVA-OneVision-1.5 共 6 个主流多模态模型，从引导成功率、语义相似度、内部概率增幅三个维度评估。^[raw/articles/emnlp-2026-multimodal-visual-concept-causal-probing-paperweekly.md]

这套设计的关键意义在于把「探针能解码出什么」升级为「改这个方向会发生什么」——相关性证据变成因果证据，这与 [[concepts/mechanistic-interpretability|机制可解释性]] 中 activation steering 的主流路线一致。^[raw/articles/emnlp-2026-multimodal-visual-concept-causal-probing-paperweekly.md]

## 核心发现：实体集中存储，抽象概念广泛分布

研究把视觉概念按由具体到抽象分为四层：**实体、视觉风格、情感、抽象概念**。干预结果分化明显：

- **情感概念最易引导**：Gemma3-4B 上引导成功率最高达 0.900。
- **抽象概念最难干预**：Qwen3-VL-32B 上成功率低至 0.160。
- **实体知识高度集中**：分布均匀度指数（DUI）平均仅 0.071，单向干预即可引发量级显著的内部概率增幅——具体物体在特定层被封装成近似「键值对」，概念向量像精准检索开关。
- **抽象概念分布广泛**：需要跨层、多方向的干预才有效，说明抽象语义没有单一「存储位点」。

^[raw/articles/emnlp-2026-multimodal-visual-concept-causal-probing-paperweekly.md]

一个直接推论：**「看得懂」和「想得通」在内部是两套机制**。模型能识别辅助线关系却算不出答案，是因为视觉感知的实体编码与逻辑推理所需的抽象表征不在同一处，感知成功不自动转化为推理能力。^[raw/articles/emnlp-2026-multimodal-visual-concept-causal-probing-paperweekly.md]

## 对多模态系统设计的启示

- **可解释性可作为回归测试**：概念方向向量与 DUI 可作为「模型是否仍以同样方式编码某类概念」的探针，用于版本迭代后的表征漂移检测。
- **干预优先选具体概念**：在需要可控生成/编辑的场景（物体增删、风格切换），单层高强度注入即可；抽象概念控制应转向提示词或工具层，而非内部干预。
- **评测不能只看识别准确率**：识别与推理的断层意味着多模态评测必须分离「感知」与「推理」两个维度，否则高分掩盖结构性缺陷。参见 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]]。

## 局限

单篇论文视角，结论基于 6 个开源 VLM 与 4 类概念的引导成功率，未覆盖闭源前沿模型；「抽象概念」的操作化定义较粗（情感/抽象合并讨论），跨语言概念是否共享同一方向向量也未验证。

## 关联

- [[concepts/mechanistic-interpretability|机制可解释性]]
- [[concepts/attention-mechanism|注意力机制]]
- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]]
- [[entities/mokus-cross-modal-knowledge-transfer|MOKUS 跨模态迁移]]（同月多模态表征研究）

→ [[raw/articles/emnlp-2026-multimodal-visual-concept-causal-probing-paperweekly|原文存档]]
