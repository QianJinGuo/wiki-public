---
title: "Qwen-Audio-3.0-TTS 发布：多语种实时语音合成模型"
created: 2026-08-31
updated: 2026-08-31
type: entity
tags: [qwen, audio, tts, speech-synthesis, multimodal, model, alibaba]
sources: [raw/articles/qwen-audio-30-tts-release-2026]
confidence: 0.8
---

# Qwen-Audio-3.0-TTS 发布：多语种实时语音合成模型

Qwen-Audio-3.0-TTS 是阿里巴巴通义千问团队发布的实时语音合成（TTS）模型，包含 Flash（低延迟，首包 ~300ms）和 Plus（高质量）两个版本。在全球第三方榜单 Artificial Analysis 中，Qwen-Audio-3.0-TTS-Plus 获得冠军。^[raw/articles/qwen-audio-30-tts-release-2026]

## 核心能力

### 多语种与方言支持

模型覆盖 16 种语言（中文、英文、日语、韩语、德语等），在 CV3-Eval multilingual benchmark 上，Flash 版本平均 WER/CER 为 3.87（16 种语言中 10 种最佳），Plus 版本为 3.96。说话人相似度方面，Plus 版本在全部 16 种语言中排名第一，平均 SS 达 82.75。^[raw/articles/qwen-audio-30-tts-release-2026]

支持 20 种中文高质量方言（广东、重庆、东北、甘肃、贵州、浙江、河北、河南、湖北、湖南、江西、宁波、宁夏、青岛、陕西、山西、山东、上海、四川、云南），通过方言强化训练阶段缓解通用语音强化学习中的"方言特色弱化"问题。^[raw/articles/qwen-audio-30-tts-release-2026]

### 自然语言指令控制

支持 Free-style 自然语言指令控制，无需专业声学知识即可通过自然语言定义情绪、角色、场景、语速等维度。支持上下文驱动的语气选择——同一句文本在不同场景下可读出不同情绪。^[raw/articles/qwen-audio-30-tts-release-2026]

### 细粒度标签控制

支持在文本中嵌入结构化标签（如 `[gasp]`、`[giggles]`、`[angry]`），对语气、情绪和呼吸等非语言细节进行精准调控，适合叙事、游戏、影视配音等场景。^[raw/articles/qwen-audio-30-tts-release-2026]

### 复杂声学鲁棒性

通过真实声学仿真训练，模型能在嘈杂环境中自动过滤干扰、保留音色。高混响和高噪声场景下的复刻清晰度和感知质量显著优于前代。^[raw/articles/qwen-audio-30-tts-release-2026]

## 技术规格

| 特性 | Flash | Plus |
|------|-------|------|
| 首包延迟 | ~300ms | 较高 |
| 定位 | 实时交互 | 高质量生成 |
| 平均 WER/CER | 3.87 | 3.96 |
| 平均 SS | 80.44 | 82.75 |
| 音频输出 | 24kHz（48K 即将上线） | 24kHz（48K 即将上线） |
| 单次合成时长 | 最长 3 分钟 | 最长 3 分钟 |

## 配套资源

精品音色库覆盖指令风格音色、20 种方言音色、细粒度控制音色以及 14+ 款小语种音色，支持开箱即用。^[raw/articles/qwen-audio-30-tts-release-2026]

## 相关实体

- [[entities/sopro-v2-on-device-tts-halo-research|SoPro V2]] — 端侧 TTS 模型
- [[entities/voxcpm2-openbmb-tts-voice-design-jikezhijia-2026-06-30|VoxCPM2]] — 开源 TTS 语音设计
- [[entities/xiaomi-dasheng-audio-foundation-model-2026|小米大圣]] — 音频基座模型
- [[entities/stable-audio-3|Stable Audio 3]] — 音频生成模型

→ [[raw/articles/qwen-audio-30-tts-release-2026|原文存档]]
