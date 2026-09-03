---
title: "TRAE SOLO移动端上线 手机也能干活了 随时随地Vibe Working 机器之心"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-05-06-TRAE-SOLO移动端上线-手机也能干活了-随时随地Vibe-Working-机器之心]
provenance_state: extracted
---

> -> [[raw/articles/2026-05-06-TRAE-SOLO移动端上线-手机也能干活了-随时随地Vibe-Working-机器之心.md|原文存档]]

sha256: 7d98f1413715e27e42cd3e7c2790f99482dcab14906f131102ecd76e4939ab38 ^[raw/articles/2026-05-06-TRAE-SOLO移动端上线-手机也能干活了-随时随地Vibe-Working-机器之心.md]

## 摘要

文章报道 TRAE SOLO 上线移动端，实现手机、桌面、网页三端打通并对 iOS/Android/Mac/Windows 全量开放，回应 Agent 时代的"数字焦虑"——离开电脑就无法掌控正在运行的长任务（对比 OpenClaw 用 IM 遥控电脑上 Claude Code 的方案：不能实时看进程、对下一步操作没有预期）。移动端与桌面端共享同一个 Agent、同一套文件系统、同一段对话上下文，且包含完整的 MTC 与 Code 功能，能力不打折：作者实测纯用手机完成了一个仿制"到点显示肥猫提醒休息"浏览器插件的项目（MTC 生成完善提示词后进 Code 构建，人只需揣着手机等通知）。更新还带来三个重量级功能：实时语音交互讨论（Agent 生成纪要、沉淀思路并下发任务，把手机变成任务起点，遗憾是语音讨论时不能联网搜索和实时执行）、飞书 CLI 接入（把飞书文档链接交给 SOLO 理解并生成方案/报告，产物以卡片沉淀）、定时任务（按固定时间或频率自动执行 Prompt，如每天上午整理 AI 产品动态生成文档）。文末提出愿景：移动互联网让服务跟着人走，智能体的移动互联让工作流摆脱工位——AI 智能体真正的主场不是 IDE 或浏览器，而是人的工作流本身。^[raw/articles/2026-05-06-TRAE-SOLO移动端上线-手机也能干活了-随时随地Vibe-Working-机器之心.md]

## 关键要点

- 三端打通：手机/桌面/网页共享同一 Agent、文件系统与对话上下文，"设备变了，心流不断"；对非开发者可把零散想法转成 PRD 草稿、运营思路或待办，对开发者可把闪念接回项目上下文成修改计划
- 移动端含完整 MTC（Make to Code）与 Code 功能：实测流程是 MTC 让智能体理解项目并输出完善提示词 markdown，再进 Code 构建浏览器插件，全程手机操作、异步等通知
- 语音交互讨论：五分钟讨论中 Agent 反应迅速并主动引导用户思维，结束时像会议软件一样输出全程总结；局限是语音讨论中无法联网搜索与实时任务执行
- 飞书 CLI 接入：理解飞书文档内容并基于上下文生成方案/报告/任务拆解，AI 创建或修改的文档以卡片形式沉淀；实测 SOLO 从混乱选题文档中整理五一假期 AI 新闻并准确提取标题
- 定时任务：让 SOLO 按固定时间/频率自动执行 Prompt 并产出结果（如每天上午自动整理 Cursor、Claude Code 等产品动态发给自己），使 Agent 成为长期主动在线的助手
- 行业判断：Cursor、GitHub Copilot、TRAE SOLO 等仍是桌面软件时代的产物；移动端把使用门槛从工程师扩展到产品、运营、管理者、测试与设计

## 来源

- 原文: [[raw/articles/2026-05-06-TRAE-SOLO移动端上线-手机也能干活了-随时随地Vibe-Working-机器之心.md|TRAE SOLO移动端上线 手机也能干活了 随时随地Vibe Working 机器之心]]
