---
title: "《语音操控 + 记忆觉醒：Hermes 高级功能完整指南》"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-05-02-语音操控-记忆觉醒-Hermes-高级功能完整指南-unknown]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/2026-05-02-语音操控-记忆觉醒-Hermes-高级功能完整指南-unknown.md|原文存档]]

sha256: 635257bf0f0908ec54acce9be47ec1c80b869e4bf7e860fb42738f68268a58ae ^[raw/articles/2026-05-02-语音操控-记忆觉醒-Hermes-高级功能完整指南-unknown.md]

## 摘要

这是一篇 Hermes 智能体高级功能实操指南，覆盖语音模式、安全机制、SOUL.md 个性化、网络与浏览器能力四大板块，并以对比表给出 Hermes 与 OpenClaw 的能力差异。语音方面，Hermes 底层依赖可本地运行的 OpenAI Whisper（模型从 tiny 约 75MB 到 large 约 3GB 五档），提供 CLI 语音模式（按住 Space 录音）、即时通信语音消息（钉钉/飞书）、语音频道（实时多轮、可打断、多人参与）三种方式，国内下载模型可设 HF_ENDPOINT=https://hf-mirror.com 镜像。安全方面内置三层防护：外发操作二次确认、桌面/下载/文档/主目录等敏感目录的高风险批量操作警告（建议配合回收站保护与分批上限）、私有信息隔离。SOUL.md（位于 [本地运行时路径已隐藏]，最大 5KB 建议 2KB 以内）包含身份、语气、边界、专业领域四个板块，与 config.yaml（技术参数）、Skills（执行规则）、MEMORY.md（项目知识）、USER.md（用户偏好）各司其职；AGENTS.md 上限 10KB、Context Files 总量上限 50KB 且加载前经恶意 prompt 注入安全扫描。网络与浏览器能力通过 MCP 协议连接 Playwright 实现登录、填表、截图等自动化，并可外接邮件、日历、网盘、文档、数据库等服务；文末结论是 Hermes 胜在个性化、隐私与复杂自动化，OpenClaw 胜在开箱即用。^[raw/articles/2026-05-02-语音操控-记忆觉醒-Hermes-高级功能完整指南-unknown.md]

## 关键要点

- 语音三种方式：CLI 语音模式（hermes --voice 或 /voice on，按住 Space 录音，可配 TTS 双向语音）、即时消息语音（钉钉/飞书发语音消息自动转写处理）、语音频道（自动检测说话起止、支持打断、共享上下文、多人参与）；依赖 ffmpeg + openai-whisper 本地安装
- Whisper 模型五档：tiny（约 75MB 最快）、base（约 140MB 推荐入门）、small（约 460MB 平衡）、medium（约 1.5GB）、large（约 3GB 最准确）；系统自带听写是零配置备选但准确率低且 Windows/macOS 在线听写会上传语音
- 安全三层防护：外发操作（邮件/消息/推文/上传）执行前展示内容并二次确认；桌面/下载/文档/用户主目录的批量删除、重命名、移动弹高风险警告；私有信息不外泄（可经 SOUL.md 加强），建议删除走回收站、批量操作设分批上限（如每次 10 个文件）
- SOUL.md 四板块：身份（可加"我不是什么"划界）、语气（禁八股模板、信息密度高、简单问题 3-5 行、代码优先）、边界（不自行提交生产分支、不存储 API Key、破坏性操作二次确认）、专业领域（分精通/熟悉/了解三档）；写作方法是"把你最受不了的 AI 行为写进去"
- 配置文件分工：SOUL.md 管"是谁"（很少修改）、config.yaml 管"用什么"（偶尔调整）、Skills 管"怎么做"（频繁更新）、MEMORY.md/USER.md 自动更新；AGENTS.md ≤10KB、SOUL.md ≤5KB、Context Files 总量 ≤50KB，超限截断并警告
- Hermes vs OpenClaw：Hermes 有三层语音方案、三层安全防护、深度 SOUL.md 个性化、Playwright 浏览器自动化、MCP 外部服务、完全本地运行，但需配置；OpenClaw 靠系统听写、仅简单角色设定、无浏览器自动化、部分依赖云端但零配置开箱即用

## 来源

- 原文: [[raw/articles/2026-05-02-语音操控-记忆觉醒-Hermes-高级功能完整指南-unknown.md|《语音操控 + 记忆觉醒：Hermes 高级功能完整指南》]]
