---
title: "Humanity's Sixth Sense (HSS) — 直觉视觉推理基准"
created: 2026-10-08
updated: 2026-10-08
type: entity
tags: [benchmark, multimodal, vision, evaluation, mllm]
sources: [raw/articles/humanitys-sixth-sense-hss-benchmark]
confidence: 0.7
provenance_state: extracted
---

# Humanity's Sixth Sense (HSS) — 直觉视觉推理基准

> **Background**：本文档基于 Scale AI（labs.scale.com）发布的 HSS 基准论文摘要页建立。因全文（arXiv/PDF）在抓取时不可达（Cloudflare 5xx + Jina 代理窗口不可达），仅存档摘要页内容。

## 核心定义

HSS（Humanity's Sixth Sense）是一个针对**直觉视觉推理**（intuitive visual reasoning）的多模态基准。现有视觉基准要么针对刻意的专家级分析，要么针对低层感知，而人类一眼完成的直觉推理（隐含的时间、空间、社会、抽象结构推断）长期缺测。HSS 覆盖图像与视频输入，采用结构化分类法组织条目，每条配有人类撰写的 prompt，探测人们一眼就能推断的隐含结构。^[raw/articles/humanitys-sixth-sense-hss-benchmark.md]

## 关键数据

- **人类 vs 前沿模型鸿沟**：人类参与者准确率 93.1%，最强模型 GPT-6-astra 在最大推理力度下仅 53.6% — 差距约 40 个百分点。^[raw/articles/humanitys-sixth-sense-hss-benchmark.md]
- **任务示例**：一眼捕捉过去成因与未来轨迹；快速一瞥判断车辆能否停进两车之间；几秒视频判断房间内谁有权威。^[raw/articles/humanitys-sixth-sense-hss-benchmark.md]
- **Agentic 设置**：对 HSS 应用动态视觉操纵（dynamic visual manipulation）的 agentic 设置可缩小但不能闭合鸿沟。^[raw/articles/humanitys-sixth-sense-hss-benchmark.md]

## 意义

HSS 将直觉视觉推理确立为可度量的能力轴，把注意力引向当前基准 scaling 尚未覆盖的能力面 — 与现有复杂任务上的强表现形成反差，说明感知-知识型 benchmark 饱和与直觉推理缺口之间存在结构性错位。^[raw/articles/humanitys-sixth-sense-hss-benchmark.md]

## 关联

- 同属 MLLM 评估体系：与 [[entities/lingbot-vision-spatial-native-vision-foundation-model-ant]]（空间原生视觉基础模型）相邻 — HSS 从评测侧探测直觉视觉推理，LingBot 从模型侧攻空间视觉。
- 与 [[concepts/verifier-paradox]] 所在的 evaluation 方法论族相邻：HSS 是"人类直觉作为天花板"的评测设计样本。
- → [[raw/articles/humanitys-sixth-sense-hss-benchmark|原文存档]]
