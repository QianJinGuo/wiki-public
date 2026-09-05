---
title: "面向 Agent 的 API 设计（Agent-Ready API）"
created: 2026-09-05
updated: 2026-09-05
type: entity
tags: [api-design, agent, mcp, agent-integration, tool-selection]
sources: [raw/articles/agent-ready-api-design-restless-2026]
confidence: 0.7
---

# 面向 Agent 的 API 设计（Agent-Ready API）

> Restless 创始人 Gregory Koberger：你的 API 曾经是一个 feature，现在它是产品本身。下一个集成你的公司不会派一个开发者去读文档——它会派一个 agent，这个 agent 既构建集成又在生产环境运行它。这改变了 API 必须承担的角色：agent 不做研究，它调用并反应，所以它需要的一切都必须出现在响应里。^[raw/articles/agent-ready-api-design-restless-2026.md]

## 设计原则

**Agent 不会去查文档。** 当它撞墙时，会猜测、尝试近似约定、依赖参数化知识，但很少主动检索文档。如果想让 agent 在你的 API 上构建，就必须把最新的文档带到它面前。^[raw/articles/agent-ready-api-design-restless-2026.md]

## 七条 Agent-Ready 实践

1. **Error body 应携带修复指引**：为 agent 编写的错误应描述如何修复，而不只是哪里出错。`invalid_parameter: date_range` 对人类都不够，对 AI 更是远远不够。示例：401 错误里直接给出 `recovery: "Mint a new token with POST /v1/tokens, then retry"`。^[raw/articles/agent-ready-api-design-restless-2026.md]
2. **征求 agent 反馈**：通过 MCP server 暴露 `agent_send_feedback` 工具，让 agent 报告它卡在哪（feature request / 缺文档 / 错误难懂），自己再分诊。^[raw/articles/agent-ready-api-design-restless-2026.md]
3. **暴露请求日志**：许多 API 问题直到生产环境用真实数据才出现。给 agent 访问实时`请求日志`的权限（如 Sentry），让它能追踪和调试自己的问题。^[raw/articles/agent-ready-api-design-restless-2026.md]
4. **限制 MCP 工具数量**：**MCP 工具选择会在约 30-50 个工具后性能退化**。大多数 API 对这个规模来说太大。最佳做法是基于用量和信号（如 `agent_send_feedback`、最常搜索的工具）自动精选一个端点短列表暴露为工具，其余通过 Tool Search 提供；也可把几个端点合并成一个 tool（Restless 称为 Usecases）。^[raw/articles/agent-ready-api-design-restless-2026.md]
5. **程序化注册 signup**：账号创建是 agent 集成断链处。新标准 `auth.md` 通过发送 token 到用户邮箱完成验证，给 agent 受作用域限定的凭据继续前进。^[raw/articles/agent-ready-api-design-restless-2026.md]
6. **为复杂 setup 提供 CLI**：好的 CLI 交互式 onboarding 但 agent 无法处理 TTY 交互。应为 CLI 构建两条流：人类一条、agent 一条。Agent 提供商会注入环境变量标识（Claude Code→`CLAUDECODE=1`，Codex→`CODEX_SANDBOX`），语言暴露 `process.stdout.isTTY`（Node）/`sys.stdout.isatty()`（Python）可检测。^[raw/articles/agent-ready-api-design-restless-2026.md]
7. **Agent-only 端点**：以前 API 是与开发者的契约；现在 API 是 agent 与产品交互的方式。允许有 agent-only 端点，随产品迭代，不断优化端点 ergonomics、移除未用的、跟上产品演进。^[raw/articles/agent-ready-api-design-restless-2026.md]

## 关键洞察

**「MCP 工具选择在 30-50 个工具后退化」** 是可迁移的通用工程事实，不只适用 Restless。这对大 tool surface 的 agent 系统设计有直接指导意义：多用 Tool Search + 精选短列表 + 多端点合成单 tool。^[raw/articles/agent-ready-api-design-restless-2026.md]

## 相关

- [[entities/anthropic-mcp-revisited|Anthropic MCP Revisited]]
- [[entities/anthropic-12-mcp-production-patterns|MCP 12 个生产模式]]
- [[entities/agent-eval-counterintuitive-insights-langfuse|Agent Eval 反直觉洞察]]

→ [[raw/articles/agent-ready-api-design-restless-2026|原文存档]]