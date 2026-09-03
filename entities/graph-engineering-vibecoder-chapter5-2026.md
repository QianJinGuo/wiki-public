---
title: "新概念 AI 编程第五章：Graph Engineering"
created: 2026-07-22
updated: 2026-08-29
type: entity
tags: ['graph-engineering', 'agent-orchestration', 'vibe-coder', 'loop-engineering', 'governance']
sources: [raw/articles/graph-engineering-vibecoder-chapter5-2026]
provenance_state: extracted
---

> -> [[raw/articles/graph-engineering-vibecoder-chapter5-2026.md|原文存档]]

- **执行图**：工作如何流动。节点=Agent/脚本/测试/审批；边=依赖/数据/分支/汇合/重试/升级 ^[raw/articles/graph-engineering-vibecoder-chapter5-2026.md]

## 摘要

本文是 VibeCoder "新概念 AI 编程"系列第五章，提出 Graph Engineering 作为五层工程栈（Prompt → Context → Harness → Loop → Graph）的最外层。Graph 分两层：执行图描述工作如何流动（节点是 Agent/脚本/测试/审批，边是依赖/数据/分支/汇合/重试/升级），治理图约束谁有权相信谁（反指标、慢层目标所有者、仲裁、独立审计）。文章给出可执行 Agent 图的设计契约——节点契约（输入含目标、验收、权限、预算，输出 artifact/evidence/state delta/failure）、边声明、四块状态（调度/artifact/context/policy），并列举集体自欺的四种模式（Goodhart、向上盲视、循环冲突、测量衰减），强调单 Loop 未做好 verifier 与硬停止之前扩图只会放大故障。^[raw/articles/graph-engineering-vibecoder-chapter5-2026.md]

## 关键要点

- 五层工程栈：Prompt Engineering → Context Engineering → Harness Engineering → Loop Engineering → Graph Engineering，外层扩大控制范围但内层依然存在
- 两层图：执行图（工作流动）+ 治理图（信任与仲裁权分配）
- 节点输出契约要求 artifact、evidence、state delta、failure，避免把自然语言丢给下游
- 四块状态：调度状态（运行位置）、artifact 状态（工件版本）、context 状态（节点所见）、policy 状态（权限规则）
- 扩图前置条件：先确保单 Loop 有外部 verifier、持久状态、成功出口、硬停止；强顺序推理、共享大块上下文、无可执行 verifier 时单 Loop 更稳更便宜
- 集体自欺四模式：Goodhart、向上盲视、循环冲突、测量衰减

## 来源

- 原文: [[raw/articles/graph-engineering-vibecoder-chapter5-2026.md|新概念 AI 编程第五章：Graph Engineering]]
- 原始链接: : "https://mp.weixin.qq.com/s/a3-nYQIezEDBFrvVXm8-bQ
