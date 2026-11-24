---
title: "FLAT: One Representation, Multiple Capabilities"
created: 2026-09-22
updated: 2026-11-09
type: entity
tags: [multimodal, architecture, representation-learning]
sources: [raw/articles/guangyusun-flat-unified-multimodal-representation]
confidence: 0.7
---

# FLAT: One Representation, Multiple Capabilities

## 摘要

FLAT 将图像与文本 resample 到同一一维连续 register token 序列，联合训练检索/描述/双向生成；nested dropout 使任意前缀成为 compute-detail 权衡——统一多模态表征的新范式^[raw/articles/guangyusun-flat-unified-multimodal-representation.md]

## 核心要点

- v*c=56（value=8, confidence=7, stars=4），newsletter ingest 2026-09-22
- 详细分析见原文存档

## 深度分析

### The modality gap is an architecture decision, not a data problem

Modern multimodal systems usually maintain separate representations for understanding (contrastive encoders in the CLIP lineage) and generation (diffusion or autoregressive decoders). FLAT instead resamples either images or text into the same one-dimensional sequence of continuous tokens: a single VLM encoder emits up to 256 continuous 64-dimensional register tokens for either modality, jointly trained for cross-modal alignment, image synthesis, and captioning. The payoff is geometric, not just architectural convenience — a single FLAT token already halves CLIP's image–text centroid distance, and the full sequence shrinks it to roughly one quarter. The likely mechanism: generation training forces the latent space to be denser and more continuous than contrastive learning alone requires, tightening alignment as a side effect. This mirrors the industry drift toward one embedding space serving both retrieval and generation, as in [[entities/gemini-embedding-2-multimodal-unified-vector-hyman|Gemini Embedding 2 的多模态统一向量]] and [[entities/douyin-dme-multimodal-embedding-multimodal-retrieval|抖音 DME 多模态检索]].

### Nested dropout turns representation length into a compute–detail dial

Nested dropout organizes information coarse-to-fine within a single encoding: the first token captures global semantics, and each additional prefix token recovers composition, spatial relations, counting, and fine-grained attributes. Because every prefix is trained to remain useful, prefix length K becomes a direct compute–detail trade-off at inference time — increasing K adds compositional detail without switching representations or models. Two details deserve attention. First, training only samples power-of-four prefix lengths, yet the ordered representation decodes smoothly at unseen values of K, so the ordering is a learned curriculum rather than a discretized lookup table. Second, the empirical envelope is unusually flat: COCO R@5 moves less than one point across a 256× range of representation width, caption quality climbs smoothly from K=1 to K=256 (40.5 BLEU-4, 138.6 CIDEr on COCO), and task-adapted prefixes reach 83.1 GenEval for text-to-image.

### One latent space that is simultaneously cross-modal, discriminative, and decodable

These three properties normally live in separate latent spaces: CLIP embeddings are cross-modal and discriminative but not decodable; VAE/diffusion latents are decodable but neither discriminative nor modality-aligned. FLAT's continuous tokens are all three at once. The frozen-feature evidence is strong: a linear classifier on frozen FLAT features reaches 73.3% ImageNet-1K top-1 with a single 64-D token (81.2% with 16 tokens, 81.8% with 64), and k-means over the frozen K=1 token recovers visually coherent ImageNet categories. Because image and text occupy a shared decodable space, latent operations become interpretable: interpolation makes image and text decoders follow corresponding intermediate concepts along the same latent trajectory, and token-wise arithmetic composes semantics (adding a full moon to a lighthouse scene) with no edit-specific training.

### Capabilities that emerge without explicit supervision

The shared encoder exposes capabilities the training mixture never directly supervised. Semantically equivalent English, Chinese, and Spanish prompts produce consistent images; emoji concepts route through the same latent space. Four sampled video frames jointly encode into one register sequence whose decoded output fuses concepts distributed across the clip rather than selecting a single keyframe. Reference-image and text-edit embeddings compose directly in latent space (44.0 Hit@1, 75.2 Hit@5). This is the classic signature of a good representation: new tasks are compositions of existing structure rather than new training runs — the same property that makes [[entities/taobao-generative-retrieval-rqvae-cikm-2026|淘宝生成式检索的 RQ-VAE 语义 ID]] reusable across ranking tasks.

## 实践启示

1. **Treat prefix length K as a serving-time compute knob.** Serve retrieval/embedding requests at tiny K (even K=1, 64 dims) and generation requests at large K from the same encoded sequence — no model switch, no re-encode.
2. **Store short prefixes in retrieval indexes.** With recall degrading less than one point over a 256× width reduction, billion-scale indexes can persist 64-D prefixes and expand on demand only for reranking or generation, cutting vector-store costs roughly proportionally.
3. **When one product needs both understanding and generation, prefer a unified continuous-token encoder over separate CLIP + generator stacks.** You inherit a smaller modality gap (centroid distance halved at K=1), one model to version, and one latency budget instead of two.
4. **Audit representation quality cheaply before investing in downstream training.** Linear probes (73.3% top-1 from one token) and k-means cluster coherence on frozen features reveal whether the space is worth building on — a 30-minute check that de-risks weeks of fine-tuning.
5. **Design features around latent composition.** Interpolation, semantic arithmetic, cross-lingual prompt consistency, and multi-frame video fusion all come free with a shared decodable space; products like style blending or concept fusion can be latent operations rather than new model fine-tunes. This is the core enabler of modern [[concepts/media-generation|media generation]] pipelines.
6. **Mind the name collision.** FLAT (this entity: resampled multimodal register tokens) shares its name with [[entities/flat-feedforward-latent-triangle-splatting|FLAT: feedforward latent triangle splatting]], a 3D rendering method — an easy mix-up when searching papers or code.

→ [[raw/articles/guangyusun-flat-unified-multimodal-representation|原文存档]]
