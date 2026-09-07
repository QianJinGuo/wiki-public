---
title: "LayerRecall：选择性长视频视频 DiT 记忆路由（浙大×港大）"
created: 2026-09-07
updated: 2026-09-07
type: entity
tags: [video-diffusion, video-dit, memory, memory-router, long-video, kv-cache, zju, hku, model-architecture, arxiv]
sources: [raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026]
confidence: 0.72
---

# LayerRecall：选择性长视频视频 DiT 记忆路由（浙大×港大）

浙江大学与香港大学团队提出 LayerRecall（arXiv 2608.28460），解决自回归视频生成模型长程记忆失败的问题：模型一段段生成画面并用 KV 缓存保存上下文，为控计算量缓存优先保留最近片段，更早的身份/属性/场景线索被挤出去。LayerRecall 不把整段历史一股脑塞回模型，而是同时回答两个问题——**现在该想起什么？这段记忆又该在哪些层里使用？**^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md]

## 两把钥匙：What to Retrieve + Where to Use

- **What to Retrieve（取什么）**：系统为历史 chunk 建立紧凑摘要作检索索引；当前画面到来后，路由器读取当前隐藏状态、形成随生成内容变化的查询，从历史候选中找出最相关记录。技术关键：摘要只负责检索打分，真正送入注意力计算的仍是被选中 chunk 的完整 K/V。
- **Where to Use（用在哪层）**：被选中的「记忆敏感层」同时看到 sink、检索历史和当前 chunk；其他层继续用原始局部滑动窗口。注意：当前状态动态控制「取什么」，「在哪里用」是预先分析并固定的骨干特定策略——不是每一步动态选择网络层。

## 反直觉发现：记忆不是越多越好，更不能每层都塞

逐层分析 LongLive-2.0 发现，有些层更依赖当前/近期画面（维持姿态、运动、局部连续性），另一些层才对远期历史投入更多注意力；且「哪一层需要长时记忆」随骨干不同而异，不能把同一组层位置当通用答案。把历史 K/V 注入所有层的 All-Layer Routing 对照显示，多了历史反而更容易出现突变和时间抖动（高频帧变化功率比 0.60→0.38）。^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md]

## 训练：Cross-Horizon Prediction Matching（CHPM）

现实中没有人逐帧标注「此刻应找回第几个 chunk」。CHPM 用同一冻结视频骨干构造「看得更远的老师」（可见 384 latent frames）与「记忆受限的学生」（单次注意力预算 32 帧）。两者面对相同噪声/文本条件/扩散时间步去噪预测差异驱动路由器学习；不要求 student 复制 teacher 注意力图。只优化 ~165 万参数记忆路由模块，5B 生成骨干/文本编码器/VAE 均冻结。随机初始化 MemoBench 0.519 → CHPM 训练后 0.548。^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md]

## 评测

MemoBench Overall 0.548（优于最佳基线 MemFlow 0.531）、MovieBench Overall 0.578（优于 Rolling Forcing 0.548）、VBench-Long 0.978（与 LongLive-2.0 骨干持平）。额外路由代价小：端到端生成 305.9s→309.4s（+3.5s），解码吞吐 5.22→5.16 FPS。^[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026.md]

> [!note] 边界：LayerRecall 并不显式检测「哪里生成错了」——展示的是与机制一致的行为案例，不是已完成因果验证的纠错模块。

→ [[raw/articles/layerrecall-memory-router-zju-hku-arxiv-2026|原文存档]]