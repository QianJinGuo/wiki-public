---
title: "啊 我刚开源的 Skills 已经 7K Star 了  code秘密花园"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-02-啊-我刚开源的-Skills-已经-7K-Star-了--code秘密花园]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/2026-06-02-啊-我刚开源的-Skills-已经-7K-Star-了--code秘密花园.md|原文存档]]

sha256: d6f064ad6fde6d1103017ed2767b9a2ecb542cb78b1817ff9e9755c5183d77c4 ^[raw/articles/2026-06-02-啊-我刚开源的-Skills-已经-7K-Star-了--code秘密花园.md]

## 摘要

作者 ConardLi 把近几篇 AI Agent 教程中陆续开源的 Skill 整理进 garden-skills 合集仓库，写文时已接近 7K Star，并借机阐述他对 Skill 价值的核心判断：Skill 的价值不在提示词漂亮，而在于把一套可重复、稳定工作的方法交给 Agent，把"任务"变成"生产线" ^[raw/articles/2026-06-02-啊-我刚开源的-Skills-已经-7K-Star-了--code秘密花园.md]。文章介绍了三个主力 Skill 及其近期更新：web-video-presentation（用网页模拟视频效果、内置多套主题模板、TTS 改为可插拔，支持 MiniMax/OpenAI 示例并兼容 ElevenLabs、edge-tts 等）、web-design-engineer（对抗"AI 味"网页审美、新增 25 套含具体设计规则的主题）、gpt-image-2（18 大类 79 个结构化 Prompt 模板、本地/宿主工具/顾问三种运行模式） ^[raw/articles/2026-06-02-啊-我刚开源的-Skills-已经-7K-Star-了--code秘密花园.md]。每个 Skill 都附有在线预览站点，作者同时给出了"模型很关键、第一轮 Review 要认真看、别期待一次到位"等使用建议 ^[raw/articles/2026-06-02-啊-我刚开源的-Skills-已经-7K-Star-了--code秘密花园.md]。

## 关键要点

- 好 Skill 的三要素：明确的工作流程（什么时候问、做、停）、明确的质量标准（什么算好、什么算 AI 味重）、明确的迭代接口（不满意时反馈什么、Agent 知道改哪一层）
- web-video-presentation 的设计动机：AI 长视频的痛点是随机抽卡与消耗爆炸，网页方案把视频拆成工程——章节、旁白、画面、主题、进度全由代码控制，可局部修改
- web-design-engineer 显式列出反模式：大渐变、玻璃卡片、发光边框、过度圆角、信息排布松散；25 套主题含 linear、raycast、aesop、tufte-dataink、y2k-retrofuturism 等
- gpt-image-2 的三种运行模式：本地模式（自备 API Key 直接出图落盘）、宿主工具模式（交给 Codex 等环境自带图像工具）、顾问模式（无图像工具时退化为 Prompt 顾问）
- 视频 Skill 效果最好的模型是 Opus 4.7；作者强调脚本、主题、章节大纲在前面定得越清楚，后面返工越少

## 来源

- 原文：[[raw/articles/2026-06-02-啊-我刚开源的-Skills-已经-7K-Star-了--code秘密花园.md|啊 我刚开源的 Skills 已经 7K Star 了  code秘密花园]]
