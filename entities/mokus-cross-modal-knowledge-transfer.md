---
title: "MoKus: Cross-Modal Knowledge Transfer for Knowledge-Aware Concept Customization"
created: 2026-07-22
updated: 2026-09-13
type: entity
tags: [multimodal, knowledge-transfer, text-to-image, ECCV-2026, diffusion, concept-customization]
sources: [raw/articles/eccv-2026-mokus打通跨模态迁移文本一改生成图像也跟着变]
confidence: 0.6
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
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

## 深度分析

### The Bridge Is a Shared Conditioning Path, Not Shared Weights
The experiment establishes something narrower than a fused knowledge base: the final hidden states of the LLM text encoder are consumed by the DiT backbone as conditioning, so any edit that lands in them is read by both modalities. Changing the answer to "Beethoven's favorite instrument" to "guitar" alters the representation that conditions generation, and the generator — holding no independent opinion about Beethoven — renders a guitar. The paper is therefore a statement about a leakage path as much as about a unified knowledge space: once an edit lands in a representation that crosses a module boundary, nothing stops it from crossing. That cuts both ways — cross-modal control becomes cheap, and so do cross-modal side effects, which is the same stress pattern studied in [[entities/llm-steering-behavior-guidance|hidden-state steering]].

### Why the Two Stages Are Load-Bearing
The framework separates an address from its content. Stage 1 binds a rare token to the concept's visual appearance through LoRA on MMDiT self-attention, making it an anchor representation that answers "what does this look like"; Stage 2 converts each natural-language statement into a query and writes the answer onto that same anchor by editing only 16 Gate and Up projection matrices in MLP layers 18–26. Because the write target is a fixed address, each further statement costs roughly seven seconds and performance stays flat from one to five, while appearance is never re-optimized. Collapsing the two stages loses exactly this economy: Naive-DB retrains DreamBooth once per statement (about 27 minutes against six), and Enc-FT fine-tunes the encoder without a structured write target, letting the edit drift the representation that also carries appearance.

### What Breaks Without the Anchor
Two failure modes justify the design. A rare-token handle carries no semantics, so language cannot retrieve it: DreamBooth-style customization reproduces the Little Mermaid sculpture from "`<sks>` sculpture" but not from the knowledge description "Little Mermaid Statue Denmark". And before any editing, the model may simply lack the fact — asked for Beethoven's favorite instrument it can generate a portrait instead, because generation is faithful to the parametric store and renders a missing fact convincingly. MoKus answers the second by writing knowledge into the store and the first by demoting the rare token to an internal index while promoting natural language to the user-facing handle.

### Where It Sits, and What the Evidence Does Not Show
MoKus is a knowledge-editing method whose edit target happens to be a representation consumed by another modality. It follows the locate-then-edit tradition — UltraEdit as the default editor, the update computed as a hidden-state direction applied to specific matrices — but its address is a visual concept rather than a fact, so one write controls naming and depiction together. It is narrower than [[entities/mind-lab-lora-continual-learning-system|LoRA-based continual adaptation]]: writing is confined to a few matrices so that appearance binding survives, which is the discipline also visible in [[entities/parametric-memory-survey-chenzikang-alitech-2026|parametric memory]] work. KnowCusBench is small and author-curated (35 concepts, five statements each, 199 prompts, five seeds), so flat stability from one to five is evidence about a narrow range, not a capacity law. A single default editor leaves sensitivity to the algorithm open, and CLIP-I-Seg, CLIP-T and PickScore measure subject fidelity, prompt adherence and predicted preference rather than factual correctness; the WISE gains are improvements on a subset with modest absolute values. Most tellingly, it reports transfer, not interference: whether editing one concept degrades other concepts or general abilities under sequential edits is unmeasured.

## 实践启示

1. **Audit the image side after any text-side edit.** Encoder hidden states are the conditioning signal, so a knowledge edit is a cross-modal write. Regression-test unrelated prompts whenever the encoder is touched, and treat unexpected image changes as expected, not as a bug.

2. **Separate the address from the content in your own customization stack.** Bind appearance to an internal token and expose natural language as the interface. You gain compositional prompts, incremental writes on the order of seconds, and training cost that no longer scales with the number of facts attached.

3. **Keep edits small and located.** Direction-based updates to a handful of MLP projection matrices preserve the rest of the model; whole-encoder fine-tuning and per-fact retraining cost far more and risk drifting the appearance representation.

4. **Measure the capacity ceiling before promising "unlimited knowledge".** Flat performance from one to five statements does not establish where knowledge writes begin to interfere. Sweep statements per concept until accuracy degrades and publish the curve.

5. **Reuse the same address for erasure and creation.** The anchor already supports concept erasure and virtual concept creation, so plan for removal requests, brand and likeness control, and licensed-concept injection on the same primitive.

6. **Evaluate facts, not just fidelity — and test interference.** Fidelity metrics cannot say whether an image is factually right or whether an edit broke a neighbouring concept. Add factual-consistency checks, non-target-concept regressions, and sequential-edit suites before trusting such a pipeline inside a [[concepts/media-generation|media generation]] product.

## Task Definition: Knowledge-Aware Concept Customization

MoKus introduces a new task where, given reference images and multiple natural language knowledge statements about a concept, the model must bind this knowledge to the concept so it can be generated in new contexts using only natural language descriptions — moving beyond the `<sks>` bottleneck toward true knowledge-driven generation. ^[raw/articles/eccv-2026-mokus打通跨模态迁移文本一改生成图像也跟着变.md]

→ [[raw/articles/eccv-2026-mokus打通跨模态迁移文本一改生成图像也跟着变|原文存档]]
