---
title: "Sopro V2: 私有、快速、设备端TTS模型"
created: 2026-08-30
updated: 2026-08-30
type: entity
tags: [tts, text-to-speech, voice-cloning, on-device, multilingual, open-source]
sources: [raw/articles/sopro-v2-on-device-tts-halo-research]
confidence: 0.8
---

# Sopro V2: 私有、快速、设备端TTS模型

> **Background**：Halo Research开源的Sopro V2系列TTS模型，其中sopro-v2-turbo是120M参数的多语言语音克隆TTS模型，支持流式输出、设备端运行，原生支持欧洲葡萄牙语。

## 模型特点

sopro-v2-turbo是120M参数的voice-cloning TTS模型，核心卖点：
- **设备端运行**：在Apple M3 CPU上RTF 0.24（离线），流式首次音频延迟~300ms
- **多语言**：英语、德语、法语、欧洲葡萄牙语（首个原生支持欧洲葡语的开源TTS）
- **极速**：单H100上RTF 0.07，流式首次音频~200ms

^[raw/articles/sopro-v2-on-device-tts-halo-research.md]

## 性能数据

| 平台 | 离线RTF | 流式首音频延迟 | 流式RTF |
|------|---------|--------------|---------|
| Apple M3 CPU | 0.24 | ~300ms | 0.21 |
| 单H100 | 0.07 | ~200ms | — |

所有数据为单流PyTorch默认设置，无batching。

## 背景故事

Sopro始于2025年12月的个人项目——Halo NeuroAI联合创始人在假期两周内构建。动机是欧洲葡萄牙语在所有商业TTS提供商（OpenAI、Cartesia、ElevenLabs）上发音不正确（巴西葡语数据偏差），且工作around增加延迟。^[raw/articles/sopro-v2-on-device-tts-halo-research.md]

V1训练成本仅$250，达到Hacker News #2，但存在不稳定、克隆质量不一致、仅支持英语等问题。V2在FCCN-FCT合作伙伴的Deucalion超算和MareNostrum 5上训练。

## 实践启示

- **设备端TTS的可行性**：120M参数模型在CPU上达到0.24 RTF，证明了高质量TTS无需云端
- **小语种覆盖**：商业TTS对小语种（欧洲葡语）的覆盖不足，开源模型填补空白
- **隐私+速度+质量三角**：设备端运行解决了隐私问题，同时延迟低于大多数云API
- **架构演进**：从V1的CSM风格（Mimi codec + autoregressive conv）到V2的改进

→ [[raw/articles/sopro-v2-on-device-tts-halo-research|原文存档]]
