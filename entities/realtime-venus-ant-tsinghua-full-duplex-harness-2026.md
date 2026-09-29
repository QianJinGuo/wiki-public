---
title: "蚂蚁清华开源 Realtime-Venus：9B 全双工交互模型 + Harness 打通前后台"
created: 2026-09-30
updated: 2026-09-30
type: entity
tags: [agent, harness, full-duplex, multimodal, open-source, realtime]
sources: [raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026]
confidence: 0.7
---

# 蚂蚁清华开源 Realtime-Venus：9B 全双工交互模型 + Harness 打通前后台

> **Background**：本文基于量子位对蚂蚁 AI 安全实验室与清华大学人机交互实验室联合发布 Realtime-Venus 的报道整理。核心信息：开源 9B 全双工交互模型（Omni/Audio 两款）+ 配套 Harness，实现"边聊边办事"的异步任务委托架构，对标 TML-Interaction-Small / SeedRealtime / Qwen-Omni-Realtime 等闭源方案。→ [[raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026|原文存档]]

## 深度分析

Realtime-Venus 解决的核心问题是实时交互的"阻塞"：查资料、跑任务需要时间，用户却在此期间继续追问或补充信息。系统将主动感知和对话控制交给前端模型，通过配套 Harness 连接后台服务，让任务执行与对话同时进行——用户可随时插话、追问，后台结果返回后模型结合对话状态选择播报时机 ^[raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026.md]。

**架构三层能力**：(1) 持续感知——模型持续接收画面和声音，在关键事件前保持倾听并主动提醒（如"裁判吹哨了"），用户交代任务后无需反复追问；(2) 全双工交互——说话时继续接收输入，结合语义判断何时续说、何时让出话语权（"对，没错"是鼓励继续，"等等，我改一下"是要求停下），被纠正时停下回应、调整未说内容再继续；(3) 后台委托——遇外部能力任务委托后台处理，结果带回对话 ^[raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026.md]。

**与 Harness Engineering 的关联**：这是少见的将 harness 概念显式应用于语音交互领域的开源实践——前端 9B 模型（感知+对话控制）与后台任务执行通过 harness 解耦，与 [[concepts/agent-harness-engineering-paradigm|Agent Harness Engineering]] 中"模型负责认知、harness 负责执行"的分层思路同构。区别在于 Realtime-Venus 的 harness 还承担全双工话语权仲裁（何时打断、何时播报），这是语音模态特有的执行控制问题 ^[raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026.md]。

**竞品坐标**：闭源阵营 TML-Interaction-Small、SeedRealtime、Qwen-Omni-Realtime 相继推出全双工方案；Realtime-Venus 以开放权重+代码的姿态入场，支持自主部署、定制与扩展，是全双工交互 + 后台任务委托能力的首个开源组合。Omni 还可借助外部记忆检索历史视听信息（回答与早先画面有关的问题），记忆检索成为长时交互的支撑组件 ^[raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026.md]。

## 相关

- [[concepts/agent-harness-engineering-paradigm|Agent Harness Engineering 范式]]
- [[concepts/embodied-intelligence-frontier|具身智能前沿]]
- [[concepts/openai-realtime-voice-architecture|OpenAI Realtime 语音架构]]

→ [[raw/articles/realtime-venus-ant-tsinghua-full-duplex-harness-2026|原文存档]]
