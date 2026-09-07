---
title: "DyRef：ECCV'26 Oral 多参考约束下的动态图像生成优化框架"
created: 2026-08-06
updated: 2026-08-06
type: entity
tags: [vision, image-generation, multi-reference, ecCV, dyref, diffusion]
sources: [raw/articles/eccv26-oral人物不能变姿势要对齐风格还得一致dyref突破多参考约束下的图像生成难题]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# DyRef：ECCV'26 Oral 多参考约束下的动态图像生成优化框架

## 问题：多参考图协调是图像生成走向专业应用的关键门槛

给图像生成模型一张人物参考图，它大概率能抓住身份特征；再给一张背景图，也许还能把人放进去。但一次给 5-7 张不同类型参考图（人物、背景、姿势、光照、风格）并要求组合成一张自然、统一、符合指令的新图时，模型可能记住人物却忘了姿势，套上背景却丢了风格，光照、构图、主体关系互相"打架"。真实设计工作（广告视觉、IP 形象延展、海报设计、视频关键帧生成）几乎总是需要同时参考多种视觉条件——这是当前图像生成走向专业应用绕不开的问题。^[raw/articles/eccv26-oral人物不能变姿势要对齐风格还得一致dyref突破多参考约束下的图像生成难题.md]

## DyRef：面向复杂多参考图像生成的动态优化框架

研究团队提出 **DyRef**，一个面向复杂多参考图像生成的动态优化框架，已被 ECCV 2026 接收为 Oral。核心思路是在多张参考图之间做动态协调，解决多约束冲突（人物一致性、姿势对齐、风格统一）。^[raw/articles/eccv26-oral人物不能变姿势要对齐风格还得一致dyref突破多参考约束下的图像生成难题.md]

## 与 Wiki 现有知识的关联

- 图像/视频生成扩散模型架构背景见 扩散模型架构
- 与 [[entities/a2rd-agentic-autoregressive-diffusion-long-video|A2RD Agentic 自回归扩散长视频]] 同属扩散生成的前沿研究
- 参考图协调问题也是多模态生成中"多条件控制"的共性挑战

→ [[raw/articles/eccv26-oral人物不能变姿势要对齐风格还得一致dyref突破多参考约束下的图像生成难题|原文存档]]
