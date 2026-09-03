---
title: "MCP vs Function Calling：Agent 工具协议对比"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, mcp, function-calling, tool-protocol, agent]
sources: [concepts/mcp-protocol-ecosystem, concepts/model-context-protocol-mcp, entities/hermes-agent]
---

## 对照表

| 维度 | MCP | Function Calling |
|------|---|---|
| 协议类型 | 客户端-服务器（HTTP/stdio） | Provider 内建 JSON Schema |
| 工具发现 | 运行时发现（list_tools） | 编译时注册 |
| 工具执行 | 服务器执行，客户端取结果 | Provider slot，本地执行 |
| 上下文注入 | MCP Resource 可注入 system | 纯函数定义，无额外上下文 |
| 跨平台 | 协议中立（任何模型可连） | Provider 限制（仅 OpenAI 兼容） |
| 生态成熟度 | 快速膨胀（2025-2026 爆发） | 成熟（OpenAI/Google/AWS 全支持） |
| Hermes 支持 | ✅ Native MCP 客户端 | ✅ 内建工具 |

## 判断

Function calling 是 Software 2.0 时代的工具协议（绑定 Provider），MCP 是 Software 3.0 时代的（解耦 Provider）。趋势：开放生态走 MCP，企业内绑定走 function calling。Hermes、Claude Code、VS Code 都已 native 支持 MCP——这是 2026 的事实标准。

## 对比方来源

- MCP protocol ecosystem
- [[concepts/model-context-protocol-mcp|model context protocol]]
- [[concepts/tool-use-patterns-ai-agents|tool use patterns]]
- [[concepts/harness-tool-design-evolution|harness 工具设计演化]]

## 进一步阅读

- "MCP 协议生态"
- [[concepts/model-context-protocol-mcp]]
- [[entities/hermes-agent]]
