---
title: "Self-Improving Agent vs Static Agent"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, self-improving, static, agent, evolution, methodology]
sources: [entities/agent-self-improvement-six-mechanisms, entities/hermes-agent-self-evolving, concepts/agent-self-improvement-loops]
---

## 对照表

| 维度 | Self-Improving | Static |
|------|---|---|
| prompt 更新 | ✅ 自动反思 → 更新 prompt/skill | ❌ 固定，最多手动调 |
| 记忆持久化 | ✅ Memory + Skills 跨 session 累积 | ❌ session 结束归零 |
| 工具扩展 | ✅ 学新工具、建新 skill | ❌ 出厂工具集固定 |
| 回滚能力 | ✅ 修改可回滚（旧 skill 保留） | N/A |
| 风险 | 中（自改进可能跑偏） | 低（不做多余的事） |
| 维护成本 | 中（初始设置 + 定期审核） | 高（每次手动调） |
| 适用场景 | 学习型 / 个人助手 / 长期协作 | 确定性任务 / SLA 严格 production |
| 代表 | Hermes Agent (Memory + Skills + Wiki) | 传统 prompt-based assistant |

## 判断

Self-improving 是个人/研究/长协作的护城河，static 是 SLA 严格生产场景的安全选择。两者可混合：核心稳定 loop 用 static，外围 skill / prompt / memory 用 self-improving。Hermes Agent 走的就是这条混合路线。

## 对比方来源

- [[concepts/agent-self-improvement-loops|agent self-improvement loops]]
- [[entities/agent-self-improvement-six-mechanisms|6 机制源]]
- [[entities/hermes-agent-self-evolving|Hermes 自进化]]
- [[concepts/ai-self-improvement-bootstrapping|self-improvement bootstrapping]]

## 进一步阅读

- [[entities/agent-self-improvement-six-mechanisms]]
- [[entities/hermes-agent-self-evolving]]
- [[concepts/agent-self-improvement-loops]]
