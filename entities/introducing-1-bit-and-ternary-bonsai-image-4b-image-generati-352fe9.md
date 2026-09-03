---

title: "Introducing 1-bit and Ternary Bonsai Image 4B: Image Generation for Local Devices"
created: 2026-06-10
updated: 2026-08-29
tags: [architecture, code, data, evaluation, fine-tuning, memory, mlops, nvidia, prompt, rl, vision, workflow]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/introducing-1-bit-and-ternary-bonsai-image-4b-image-generati-352fe9
---

# Introducing 1-bit and Ternary Bonsai Image 4B: Image Generation for Local Devices

→ [[raw/articles/introducing-1-bit-and-ternary-bonsai-image-4b-image-generati-352fe9|原文存档]] ^[raw/articles/introducing-1-bit-and-ternary-bonsai-image-4b-image-generati-352fe9.md]

## 深度分析

Introducing 1-bit and Ternary Bonsai Image 4B: Image Generation for Local Devices 涉及architecture领域的核心技术议题。 ^[raw/articles/introducing-1-bit-and-ternary-bonsai-image-4b-image-generati-352fe9.md]
### 核心观点
1. Bonsai Image 4B comes in two variants: ^[raw/articles/introducing-1-bit-and-ternary-bonsai-image-4b-image-generati-352fe9.md]
*   **1-bit Bonsai Image 4B** uses binary {−1, +1} transformer weights with an FP16 group-wise scaling factor, giving 1.
2. 125 effective bits per weight. ^[raw/articles/introducing-1-bit-and-ternary-bonsai-image-4b-image-generati-352fe9.md]
3. It targets maximum compression and is the right fit when memory pressure, bandwidth, and the deployment footprint are the primary constraints. ^[raw/articles/introducing-1-bit-and-ternary-bonsai-image-4b-image-generati-352fe9.md]
4. *   **Ternary Bonsai Image 4B** uses {−1, 0, +1} transformer weights with an FP16 group-wise scaling factor, giving 1. ^[raw/articles/introducing-1-bit-and-ternary-bonsai-image-4b-image-generati-352fe9.md]
5. 71 effective bits per weight. ^[raw/articles/introducing-1-bit-and-ternary-bonsai-image-4b-image-generati-352fe9.md]

### 关联实体

- [[entities/scale-robot-reinforcement-learning-with-nvidia-isaac-lab-on-]]
- [[entities/nvidia-isaac-lab-sagemaker-robot-rl-humanoid]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/ethan-he-cosmos-grok-imagine-latent-space-video-agent-20260606]]
- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]

## 相关实体

- [[moc/data-infrastructure|MOC]]
