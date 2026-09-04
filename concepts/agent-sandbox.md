---
title: Agent 沙箱与执行容器
created: 2026-09-05
updated: 2026-09-05
type: concept
tags: [sandbox, agent, security, isolation, execution-container, harness]
confidence: 0.75
provenance_state: merged
---

# Agent 沙箱与执行容器（Agent Sandbox & Execution Containers）

> 库内 38 个 `sandbox` 标签实体（2026-09-05 概念缺位检测）的核心主题：Agent 代码执行的隔离基底。此前没有 concept 页收拢。建立于 [[drafts/wiki-emergent-viewpoints-2026-09-concept-gap|2026-09 概念缺位透镜轮]]。

## 为什么沙箱是 2026 的独立概念层

Agent 从"生成文本"变成"执行代码"后，执行环境本身成为产品差异点与攻击面。库内簇显示三条独立的演进线，共享同一组词汇（isolation / escape / boundary）却分布在安全、harness、企业三个话题域：

**线 1 —— 执行容器的产品化**：[[entities/microsoft-mxc-execution-containers-agent-sandbox-origin|Microsoft MXC（eXecution Containers）]] 用 AppContainer/Sandboxie 系机制做跨 OS 代理代码执行容器^[raw/articles/microsoft-mxc-execution-containers-agent-sandbox-origin.md]，[[entities/mxc-execution-containers-internals-origin|其 internals 页]]拆到隔离原语层；[[entities/claude-managed-agents-self-hosted-sandbox-enterprise|Claude Managed Agents 企业边界更新]]把自托管沙箱作为企业采用的前置条件。共同点：沙箱从"开发者工具"升格为"企业采购项"。

**线 2 —— 逃逸作为攻防基准**：[[entities/nomshub-cursor-remote-tunnel-sandbox-breakout-straiker|NomShub——Cursor 远程隧道沙箱逃逸链]]代表攻击侧：Shell builtin 绕过 + 隧道外联的组合让"沙箱内"不等于"安全内"。逃逸链与 [[concepts/agent-security-attack-defense|Agent 攻防]] 簇的注入类攻击互补：注入攻的是模型，逃逸攻的是执行层隔离。

**线 3 —— harness 对沙箱的依赖**：Deep Research 类脚手架把沙箱当作默认执行基元（[[entities/miroflow-deep-research-agent-harness-mirothinker|MiroFlow/MiroThinker]]），浏览器原生 harness（[[entities/peerd-browser-native-agent-harness|peerd]]）则把沙箱语义带进浏览器进程模型。自进化系统的写入闸门（[[entities/self-evolving-agent-security-governance-hermes|3 类污染风险与 5 道写入闸门]]）本质是"对 Agent 自身产物也用沙箱语义"——隔离对象从执行环境扩展到记忆/技能写入。

## 最小判定框架

评估任一 Agent 沙箱方案的四问（从簇内页面归纳）：

1. **隔离原语是什么**：VM / microVM / 容器 / OS 级（AppContainer） / 浏览器进程——逃逸面完全不同
2. **网络策略**：能否限制外联（NomShub 案的隧道外联是常见失守点）
3. **文件系统边界**：只读基座 + 显式写入区的粒度
4. **与 harness 的耦合**：沙箱是外挂组件还是执行原语（MXC 路线 vs peerd 路线）

## 检索入口

- 高分簇页：[[entities/microsoft-mxc-execution-containers-agent-sandbox-origin|MXC]] · [[entities/claude-managed-agents-self-hosted-sandbox-enterprise|Managed Agents 企业边界]] · [[entities/nomshub-cursor-remote-tunnel-sandbox-breakout-straiker|NomShub 逃逸链]] · [[entities/miroflow-deep-research-agent-harness-mirothinker|MiroFlow]]
- 相邻概念：[[concepts/agent-security-attack-defense|Agent 攻防]] · [[concepts/agent-security-architecture|Agent 安全架构]] · [[concepts/harness-engineering-framework|Harness Engineering 框架]]
- 检测数据：[[queries/vault-evolution-dashboard|进化仪表板]] · `metrics/concept-gaps.json`
