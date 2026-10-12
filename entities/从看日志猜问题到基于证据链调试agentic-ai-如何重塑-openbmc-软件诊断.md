---
title: "字节 STE：OpenBMC 证据链 Agentic 调试"
created: 2026-10-12
updated: 2026-10-12
type: entity
tags: [agent, debugging, program-analysis, harness]
sources: [raw/articles/从看日志猜问题到基于证据链调试agentic-ai-如何重塑-openbmc-软件诊断]
confidence: 0.75
---

# 字节 STE：OpenBMC 证据链 Agentic 调试

## 深度分析

字节跳动 STE 固件团队在 OSFC2026 分享的 OpenBMC Agentic 调试方案：不让 LLM 直接读海量日志猜根因（上下文成本高 + 结论不可验证），而是先用确定性程序分析构建可验证证据链——把运行时日志从「文本线索」转为「程序足迹」（调用图、正常/异常模式），再让 Agent 在受约束的证据空间中完成根因解释、验证建议与修复方向生成。LLM 的位置被重定义为证据解释器、假设组织者与行动建议生成器，事实来源是程序分析。核心痛点对位：最后一条错误日志不一定是根因、文本相似不等于因果相关。 ^[raw/articles/从看日志猜问题到基于证据链调试agentic-ai-如何重塑-openbmc-软件诊断.md]

## 关联

- [[concepts/agent-harness-engineering-paradigm|Harness Engineering]]

→ [[raw/articles/从看日志猜问题到基于证据链调试agentic-ai-如何重塑-openbmc-软件诊断|原文存档]]
