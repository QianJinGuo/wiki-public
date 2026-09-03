---
title: "NanoGPT Speedrun MUDD 优化：彩云科技连续两次刷新训练速度世界纪录"
created: 2026-09-02
updated: 2026-09-02
type: entity
tags: [training-optimization, gpu-kernel, nanoGPT, speedrun, architecture, skip-connections, MUDD, muon, pretraining]
sources: [raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录]
confidence: 0.7
---

# NanoGPT Speedrun MUDD 优化：彩云科技连续两次刷新训练速度世界纪录

## 背景

NanoGPT Speedrun 是 GitHub 上的开源训练竞速项目，参赛者使用固定 8 张 H100 将 GPT-2 Small 规模模型训练到 loss ≤ 3.28（FineWeb 验证集），比较完整训练耗时。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md]

该项目是大模型训练技术的试验场——Muon 优化器最早在此验证后被 Kimi 用于万亿参数 K2 训练，DeepSeek 的 Engram 条件记忆也启发了榜单中的 Bigram Hash Embedding。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md]

## 彩云科技的两次突破

### 第一刀：MUDD Skip Connections（#81）

彩云科技基础模型算法团队提交 #81 MUDD Skip Connections，将训练时间从 84.36 秒压到 81.78 秒（缩短 3.1%）。

**核心思想**：传统 Transformer 残差连接是固定路径，不管处理什么类型的 token 都沿同一路线传递信息。MUDD（Multiway Dynamic Dense Connections）让不同 token 根据自身需求动态选择信息传递路径——有的 token 需要回头获取前几层的词法信息，有的更需要最近几层的推理结果。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md]

### 第二刀：MUDD Gates + DC MHA（#85）

一个多月后，#85 MUDD Gates + DC MHA 再次将时间从 79.2 秒降到 76.3 秒（缩短约 3.7%）。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md]

两次提交合计将训练时间从 84.36 秒压缩到 76.3 秒，总提速约 9.5%。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md]

## 技术意义

- **动态路由 vs 固定残差**：MUDD 的核心贡献是证明了在小模型训练中，动态稠密连接比固定残差连接更高效，为大模型训练架构优化提供了实证
- **NanoGPT 作为技术验证场**：榜单上的优化技术（Muon、MUDD 等）会被验证后迁移到大规模训练中，具有前瞻指示意义
- **低成本高影响力**：3 人小团队在固定硬件约束下取得突破，证明架构创新仍有显著空间

→ [[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录|原文存档]]
