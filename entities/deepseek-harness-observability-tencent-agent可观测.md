---

title: "DeepSeek Harness 规模化踩坑实录：Agent 可观测方案"
created: 2026-08-30
updated: 2026-08-30
type: entity
tags: [harness, observability, deepseek, agent, monitoring, tracing]
sources: [raw/articles/deepseek-harness-observability-tencent-agent可观测]
confidence: 0.8
---

# DeepSeek Harness 规模化踩坑实录：Agent 可观测方案

腾讯云日志服务团队分享了 DeepSeek Harness（DSH）规模化使用中的可观测经验，提供了全景 Agent 可观测方案。 ^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]

## DeepSeek Harness 运行原理

DSH 是 DeepSeek 开源的编码 Agent 框架，采用 **Cordis 律内核 + 全插件化** 架构：
- 内核只负责按 profile 装配插件与管理生命周期
- 模型适配器、工具集、沙箱策略、会话持久化均为可插拔插件
- web / tui / headless 三种形态共用同一个内核 ^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]

## 规模化三大痛点

1. **耗时分析**：耗时花在模型推理还是工具执行上？
2. **成本追踪**：Token 消耗集中在哪些会话、哪些模型上？
3. **失败回溯**：失败与中断发生在哪一步，过程是否可以回溯？ ^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]

## 可观测方案

腾讯云 Agent 可观测在 OneSuite 基础上提供了 DSH 采集插件：
- 以原生插件形态挂载，将执行过程还原为结构化调用链
- 覆盖**任务与推理轮次、模型调用、工具调用、会话关联**
- 配合链路检索、聚合分析与告警仪表盘 ^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]

## 关键洞察

DSH 自带会话轨迹视图、Session 事件流落盘与工具调用检索，但作用域限于**本机、单会话、实时**。跨会话、跨机器的长期留存需要额外的可观测基础设施。这是 Harness 规模化从"单机开发"到"团队协作"的关键跃迁。 ^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]

→ [[raw/articles/deepseek-harness-observability-tencent-agent可观测|原文存档]] ^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]