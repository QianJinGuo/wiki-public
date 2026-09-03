---
title: "AI Kuikly 7 5小时落地三端 多模态聊天 App 实战 腾讯技术工程"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-30-AI-Kuikly-7-5小时落地三端-多模态聊天-App-实战-腾讯技术工程]
provenance_state: extracted
---

> -> [[raw/articles/2026-06-30-AI-Kuikly-7-5小时落地三端-多模态聊天-App-实战-腾讯技术工程.md|原文存档]]

sha256: 8b209511f1562cf2927f98551fff883a069ccea2db5dade11d4dc8f63644cd6c ^[raw/articles/2026-06-30-AI-Kuikly-7-5小时落地三端-多模态聊天-App-实战-腾讯技术工程.md]

## 摘要

腾讯 Kuikly 团队的实战记录：一位开发者仅凭自然语言，用 28 轮对话、740 字 prompt 生成约 3500 行代码，7.5 小时交付一套覆盖 Android、iOS、鸿蒙三端的多模态 AI 聊天 App，全程零手写代码，而传统三端各写一遍需 30 人天、Kuikly 手写也需 7.5 人天 ^[raw/articles/2026-06-30-AI-Kuikly-7-5小时落地三端-多模态聊天-App-实战-腾讯技术工程.md]。交付物具备流式 Markdown、拍照识图、相册选取、SSE 长连接、本地会话管理等完整能力，非截图 Demo 而是三端真机可运行的应用 ^[raw/articles/2026-06-30-AI-Kuikly-7-5小时落地三端-多模态聊天-App-实战-腾讯技术工程.md]。文章把提效归因于"AI + Kuikly"组合：Kuikly AI 的 Skills 和 Rules 把框架知识喂给模型，让 AI 像资深 Kuikly 开发者一样工作——知道该用 KuiklyChatUI、AiMessageText 等现成组件而不造轮子，用 kuikly-expand-api 技能补齐 SSEModule 与 ImageModule 两个跨端 Module，遇到相册缩略图加载失败时能自己加日志、用 logcat + adb 把根因定位到 ImageAdapter 缺少 content:// URI 支持 ^[raw/articles/2026-06-30-AI-Kuikly-7-5小时落地三端-多模态聊天-App-实战-腾讯技术工程.md]。作者的结论是 Kuikly 消灭跨端重复劳动、AI 消灭框架样板劳动，人只留下产品判断、体验打磨和工程把关 ^[raw/articles/2026-06-30-AI-Kuikly-7-5小时落地三端-多模态聊天-App-实战-腾讯技术工程.md]。

## 关键要点

- 一天时间线：09:00-09:10 环境准备（含 npx skills add Tencent-TDS/KuiklyUI-AI/skills）→ 09:10-10:20 用 CodeBuddy brainstorming 技能做需求分析与组件调研 → 10:20-11:10 编码 → 11:10-12:30 真机自测 → 14:00-17:30 迭代优化 → 17:30-18:00 验收
- 组件调研细节：AI 原计划引入 KuiklyMarkdown，发现 KuiklyChatUI 的 AiMessageText 已覆盖 AI 消息 Markdown 渲染后放弃单独引入，最终落地 6 个组件——这是 Skills 和 Rules 带来的"知道什么时候不该写"
- 自动定位的 Bug：相册缩略图空白，根因是 KuiklyAlbum 给出的 content:// 格式地址不被模板工程默认 ImageAdapter 识别（只支持 base64、http、assets、file）
- 迭代中的典型问题：键盘遮挡输入框（AI 自己发明零尺寸代理 Input 承接键盘事件的绕法）、鸿蒙端新建会话不生效（RouterAdapter 未考虑先 openPage 再 closePage 的边界）、宫格快捷面板与键盘抬升互斥导致输入框跳变
- 文中提及的对照案例：搜狗输入法用 Spec Coding 把新页面开发从 3 天压到 1 天，QQ音乐用 AI 智能转码实现 90%+ 代码采用率

## 来源

- 原文：[[raw/articles/2026-06-30-AI-Kuikly-7-5小时落地三端-多模态聊天-App-实战-腾讯技术工程.md|AI Kuikly 7 5小时落地三端 多模态聊天 App 实战 腾讯技术工程]]
