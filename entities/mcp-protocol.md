---
title: "MCP Protocol"
created: 2026-07-27
updated: 2026-08-01
type: entity
tags: ["mcp", "protocol", "agent"]
sources: [raw/articles/mcp-tool-design-practical-approaches-and-tradeoffs]
provenance_state: extracted
confidence: 0.6
---

# MCP Protocol

> -> [[raw/articles/mcp-tool-design-practical-approaches-and-tradeoffs.md|原文存档]]

## 概述

When [Model Context Protocol (MCP)](<https://modelcontextprotocol.io/>) tools underperform, the cause is rarely the protocol itself but the tool design. Many teams start by exposing an existing API as-is and trusting the agent to figure out the rest. It is a natural way to extend APIs to agentic systems and generative AI coding tools. For straightforward use cases, it can work. But often it does not. You must design your tools for how large language models (LLMs) and agentic systems work. Without this, you risk failed tool calls, wrong parameter values, and retries that waste context and degra... ^[raw/articles/mcp-tool-design-practical-approaches-and-tradeoffs.md]

## 主要内容

- Approaches and tradeoffs
- Descriptions and responses
- Schema constraints
- Restructuring tools and on-demand context
- Server-side inference
- Agentic tools
- Walkthrough
- Prerequisites

## 来源

- [[raw/articles/mcp-tool-design-practical-approaches-and-tradeoffs.md|原文存档]]
- 原始链接: https://aws.amazon.com/blogs/machine-learning/mcp-tool-design-practical-approaches-and-tradeoffs
