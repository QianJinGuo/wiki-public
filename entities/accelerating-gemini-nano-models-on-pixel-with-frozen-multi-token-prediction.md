---
title: "Accelerating Gemini Nano models on Pixel with frozen Multi-Token Prediction"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/accelerating-gemini-nano-models-on-pixel-with-frozen-multi-token-prediction]
provenance_state: extracted
---

> -> [[raw/articles/accelerating-gemini-nano-models-on-pixel-with-frozen-multi-token-prediction.md|原文存档]]

sha256: 233a03ce4ae5ce97030be14168b6ad0ca02d8154704060ce323fecaf90e1db1d ^[raw/articles/accelerating-gemini-nano-models-on-pixel-with-frozen-multi-token-prediction.md]

## 摘要

Google Research 宣布一种将 Multi-Token Prediction（MTP）改装到已部署"冻结" Gemini Nano v3 模型上的新架构，已推向 Pixel 9 和 10 系列，为 AI Notification Summaries 与 Proofread 等端侧功能带来开箱即用的提速 ^[raw/articles/accelerating-gemini-nano-models-on-pixel-with-frozen-multi-token-prediction.md]。相比传统 speculative decoding 需要独立的小型 drafter 模型（与主模型争抢内存、看不到主模型内部状态），MTP 在冻结权重的主模型末层外挂一个轻量 Transformer head，利用主干已算好的高维激活自回归预测未来 token；由于错误草稿会在验证阶段被丢弃，最终输出与原模型逐位一致，能力与安全对齐零退化 ^[raw/articles/accelerating-gemini-nano-models-on-pixel-with-frozen-multi-token-prediction.md]。针对移动端内存瓶颈，团队设计了 zero-copy 架构：MTP head 不维护自己的 KV cache，直接交叉注意主模型冻结的 KV cache，省去 drafter prefill 延迟并节省每实例 130MB 内存 ^[raw/articles/accelerating-gemini-nano-models-on-pixel-with-frozen-multi-token-prediction.md]。实测中 MTP drafter 预测更准，Pixel 9 上相对同等参数量的独立 drafter 提速 50% 以上，结构可预测任务（如智能回复）的 token 接受率最高提升 55%；生产工作负载中每次推理平均多预测近 2 个 token，并因更少唤醒重处理器而降低能耗 ^[raw/articles/accelerating-gemini-nano-models-on-pixel-with-frozen-multi-token-prediction.md]。

## 关键要点

- 传统自回归生成逐 token 输出是移动端瓶颈：手机有严格能耗预算和内存上限，逐字生成低利用率地占用处理能力与内存带宽
- speculative decoding 两段式：小 drafter 生成候选（如 3 个 token）→ 大模型并行验证，不匹配则回滚到首个分歧点；独立 drafter 的缺陷是与主模型争抢 RAM 且对主模型内部状态"失明"
- 冻结骨干的优势：只训练 MTP head 参数最小化未来 token 预测误差，MTP 成为纯粹的效率优化，保持向后完全兼容
- zero-copy 细节：省去 drafter 的 embedding 查找表、prefill 点积注意力变体和应用专属调参，每实例节省 130MB；同时消除 drafter prefill 延迟
- 未来方向：并行解码与无辅助头范式、允许模型并行探索分支可能性的技术、对特定用例放宽草稿与验证的严格逐 token 匹配（verification leniency）

## 来源

- 原文：[[raw/articles/accelerating-gemini-nano-models-on-pixel-with-frozen-multi-token-prediction.md|Accelerating Gemini Nano models on Pixel with frozen Multi-Token Prediction]]
