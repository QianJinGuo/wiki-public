---
title: 媒体生成（Media Generation）
created: 2026-09-05
updated: 2026-09-05
type: concept
tags: [media-generation, video-generation, diffusion, image-generation, generative-models]
confidence: 0.7
provenance_state: merged
---

# 媒体生成（Media Generation）

> `video-generation`(42) 与 `diffusion`(54) 两簇的合并收拢页（2026-09 概念缺位第二轮批量消化，见 [[drafts/wiki-emergent-viewpoints-2026-09-metrology|度量轮]]）。两簇共享技术底座（扩散/流匹配 + 条件控制），按产出模态与用途分四个子域。

## 子域地图

**1. 海报/视觉设计 AIGC（生产管线最成熟）**：[[entities/meituan-poster-aigc-postercraft-posteromni-posterreward-meigen|美团海报 AIGC 技术体系]]（PosterCraft/PosterOmni/PosterReward 全链路）代表"垂直场景 + 业务约束 + 奖励模型"的工业化形态——生成质量由业务验收反推，不由美学评分。

**2. 长视频一致性（当前最难的工程问题）**：[[entities/joyai-echo-long-video-framework-jd|京东 JoyAI-Echo]] 攻"5 分钟一致性 + 7.5x DMD 加速 + Director Agent"——长视频的瓶颈不是单帧质量而是跨镜头一致性，Director Agent 的出现说明**生成管线正在 agent 化**（导演作为编排层）。

**3. 具身/物理视频（与世界模型交叉）**：[[entities/fine-tuning-cosmos|Cosmos 微调]]、[[entities/molmomotion-language-guided-3d-motion-forecasting|MolmoMotion 3D 运动预测]]——生成模型作为物理模拟器，服务机器人训练而非观看。此子域归 [[concepts/world-models|世界模型]]管，本页只做接口。

**4. 编辑与修复（轻量化方向）**：[[entities/moebius|Moebius: 0.2B 轻量图像修复]]代表"小模型 + 窄任务"路线——与"大模型万能"相反的工程判断：编辑任务的分布足够窄，0.2B 就够。

## 跨子域的两条主轴

- **一致性 vs 多样性**：海报要品牌一致、长视频要跨镜头一致、编辑要保真一致——"一致性工程"是媒体生成区别于纯研究的核心难题；-trajectory 级方案（[[entities/ntm-normalizing-trajectory-models|Normalizing Trajectory Models]]）从建模侧回应同一问题。
- **验收权移交**：AIGC 落地的验收标准从 FID/FVD 转向业务方（海报:点击率；视频:可用镜头比例）。与 [[concepts/eval-optimizer-firewall|评测防火墙]]的张力：业务验收指标一旦被生成器针对性优化，会重演 Goodhart。

## 检索入口

高分簇页：[[entities/meituan-poster-aigc-postercraft-posteromni-posterreward-meigen|美团海报]] · [[entities/moebius|Moebius]] · [[entities/joyai-echo-long-video-framework-jd|JoyAI-Echo]] ｜ 相邻：[[concepts/world-models|世界模型]] · [[concepts/embodied-intelligence-frontier|具身智能]]
