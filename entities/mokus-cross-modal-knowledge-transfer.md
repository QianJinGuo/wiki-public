---
title: "MoKus: Cross-Modal Knowledge Transfer for Knowledge-Aware Concept Customization"
created: 2026-07-22
updated: 2026-08-29
type: entity
tags: [multimodal, knowledge-transfer, text-to-image, ECCV-2026, diffusion, concept-customization]
sources: [raw/articles/eccv-2026-mokus打通跨模态迁移文本一改生成图像也跟着变]
confidence: 0.6
---

# MoKus: Cross-Modal Knowledge Transfer

MoKus (Leveraging Cross-Modal Knowledge Transfer for Knowledge-Aware Concept Customization) is a **ECCV 2026** paper from Tsinghua University and HKUST that demonstrates a phenomenon called **cross-modal knowledge transfer** in unified multimodal models: updating knowledge in the text modality can directly affect image generation outputs. ^[raw/articles/eccv-2026-mokus打通跨模态迁移文本一改生成图像也跟着变.md]

## Core Observation

The researchers found that in unified models capable of both understanding and generating images, textual knowledge edits propagate through the model's shared representational space. For example, after editing the LLM text encoder to change "Beethoven's favorite instrument" from "piano" to "guitar", generating an image with the description "the favorite instrument of Ludwig van Beethoven" would show a guitar — even though only the text-side knowledge was modified. This provides an empirical test for the hypothesis of a **unified knowledge space** across modalities. ^[raw/articles/eccv-2026-mokus打通跨模态迁移文本一改生成图像也跟着变.md]

## Two-Stage Framework

### Stage 1: Visual Concept Learning
Uses a rare token (like `<sks>`) as an **anchor representation** for the target concept's visual appearance. The model fine-tunes MMDiT self-attention layers via LoRA to bind the anchor token to visual details. Unlike traditional [[rag-retrieval-augmented-generation|concept customization]] approaches like DreamBooth, the rare token here serves as an internal index rather than the user-facing concept name. ^[raw/articles/eccv-2026-mokus打通跨模态迁移文本一改生成图像也跟着变.md]

### Stage 2: Textual Knowledge Updating
Natural language knowledge statements are converted to queries, and their answers are updated onto the anchor representation. Using UltraEdit as the knowledge editing method, only 16 parameter matrices in the LLM encoder's MLP layers (layers 18-26, Gate and Up projections) are modified. This separates "what the concept looks like" (anchor) from "how it can be referred to" (natural language knowledge). ^[raw/articles/eccv-2026-mokus打通跨模态迁移文本一改生成图像也跟着变.md]

## Task Definition: Knowledge-Aware Concept Customization

MoKus introduces a new task where, given reference images and multiple natural language knowledge statements about a concept, the model must bind this knowledge to the concept so it can be generated in new contexts using only natural language descriptions — moving beyond the `<sks>` bottleneck toward true knowledge-driven generation. ^[raw/articles/eccv-2026-mokus打通跨模态迁移文本一改生成图像也跟着变.md]

→ [[raw/articles/eccv-2026-mokus打通跨模态迁移文本一改生成图像也跟着变|原文存档]]
