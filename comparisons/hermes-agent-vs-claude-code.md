---
title: "Hermes Agent vs Claude Code：Agent 框架架构对比"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, hermes-agent, claude-code, agent, framework, architecture]
sources: [entities/hermes-agent, entities/hermes-agent-self-evolving, entities/claude-code-core-internals, entities/claude-code-architecture]
---

## 对照表

| 维度 | Hermes Agent | Claude Code |
|------|---|---|
| 定位 | CLI AI 代理 + 用户代理（浏览器/电话/SMS） | 终端编码 Agent |
| 部署形态 | 本地 CLI + 消息平台 Gateway | 终端 CLI，可被 VSCode 等调用 |
| 记忆系统 | Memory + Skills + Wiki 三层互喂 | 临时 working memory + 文件系统状态 |
| 工具扩展 | Skill 系统 + MCP 客户端 | 内建 20+ 工具，可文件系统扩展 |
| 多智能体 | delegate_task (N ≤ max_concurrent) | Task tool（subagent spawn） |
| 自动化 | cronjob 系统 + auto-evolve | Auto Mode + Review Mode |
| 开源 | ✅ MIT | ❌ 闭源（Binary 分发） |
| 平台覆盖 | macOS + Linux 为主 | macOS + Linux + Windows (WSL) |

## 判断

两者定位不同——Hermes 是「通用 AI 代理 + 多平台交付」，Claude Code 是「专做编码的终端 Agent」。Hermes 的护城河在 Memory+Skills+Wiki 三层记忆系统和 cronjob；Claude Code 的护城河在 Anthropic 模型集成和 tool 设计成熟度。对开发者：编码任务用 Claude Code，全栈个人助手用 Hermes，两者可通过 MCP 互相调用。

## 对比方来源

- [[entities/hermes-agent|Hermes Agent 主页]]
- [[entities/hermes-agent-self-evolving|Hermes Agent 自进化]]
- [[concepts/hermes-agent|Hermes Agent concept]]
- [[concepts/claude-code-deep-architecture-analysis|Claude Code 架构]]

## 进一步阅读

- [[entities/hermes-agent]]
- [[entities/hermes-agent-self-evolving]]
- [[entities/claude-code-core-internals]]
- [[entities/claude-code-architecture]]
