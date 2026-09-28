---
title: "Agent Skill 规范、构建与设计模式"
type: entity
created: 2026-07-02
updated: 2026-09-29
tags: [agent, skill, specification, design-patterns, anthropic, google-adk, skill-creator]
rating: v7c7
sources:
  - raw/articles/agent-skill-spec-building-design-patterns
  - raw/articles/vercel-agent-plugins-skill-mcp-packaging-2026
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Agent Skill 规范、构建与设计模式

基于 Anthropic Agent Skills 规范、Skill-Creator 方法论、Superpowers Writing-Skills 框架及 Google ADK 设计模式的系统性总结。 ^[raw/articles/agent-skill-spec-building-design-patterns.md]

## 核心概念

**Skill ≠ Prompt**：Skill 是围绕任务、工具、流程和输出边界的结构化行为设计，是可复用的 Prompt 增强包。^[raw/articles/agent-skill-spec-building-design-patterns.md]


### SKILL.md 格式标准（Anthropic 2025.12）
- `SKILL.md`：YAML 元数据 + Markdown 指令
- `scripts/`：可执行脚本
- `references/`：按需加载的参考文档
- `assets/`：模板、资源文件

### 命名规则
仅允许 Unicode 小写字母、数字和连字符，不能以连字符开头/结尾。^[raw/articles/agent-skill-spec-building-design-patterns.md]

## 三层渐进式加载机制

解决上下文膨胀问题的核心机制，借鉴 UI/UX 渐进式信息披露策略：^[raw/articles/agent-skill-spec-building-design-patterns.md]


| 层级 | 内容 | 加载时机 | Token 成本 |
|------|------|----------|-----------|
| **L1 目录层** | name + description | 会话启动时 | ~50-100 tokens/个 |
| **L2 指令层** | 完整 SKILL.md body | Skill 被激活时 | 建议 <5000 tokens |
| **L3 资源层** | scripts/references/assets | 指令引用时按需 | 视文件大小 |

即使安装 20 个 Skill，初始加载仅 1000-2000 tokens，上下文使用量减少约 **90%**。^[raw/articles/agent-skill-spec-building-design-patterns.md]

### 触发机制
完全由模型自行判断当前任务是否匹配 description，非关键词硬编码。^[raw/articles/agent-skill-spec-building-design-patterns.md]


**最关键发现**：Description 只应描述触发条件，绝不要总结工作流程——否则 Agent 会直接按 description 执行，跳过读取完整的 SKILL.md 正文。^[raw/articles/agent-skill-spec-building-design-patterns.md]

## Skill-Creator（Anthropic）工程化方法论

核心思想：像做机器学习一样做 Prompt Engineering。^[raw/articles/agent-skill-spec-building-design-patterns.md]


### 三大核心思想
- **泛化而非过拟合**：不为测试用例做针对性修改
- **解释"为什么"而非堆砌"必须"**：LLM 有良好的心智理论，解释比命令更有效
- **提取重复模式**：Agent 反复写的辅助脚本应抽取到 `scripts/` 目录

### 三 Agent 专业化评估链
- **Grader（评分者）**：评估断言，且会自我批评
- **Comparator（盲比较者）**：双盲实验，不知哪个输出对应哪个 Skill
- **Analyzer（分析者）**：事后揭盲分析赢家为什么赢^[raw/articles/agent-skill-spec-building-design-patterns.md]

## 五大设计模式（Google ADK）

| 模式 | 核心逻辑 | 适用场景 |
|------|----------|----------|
| **Tool Wrapper** | SKILL.md 不写完整规范，只告诉 Agent 去 references/ 按需加载 | 框架/库封装、团队编码规范 |
| **Generator** | 模板 + 风格指南 + 主动提问 | 标准化文档生成、项目脚手架 |
| **Reviewer** | 分离"查什么"与"怎么查"，解释 WHY 不是 WHAT | 自动化 PR 审查、安全扫描 |
| **Inversion** | 翻转交互模式：Agent 先采访用户，再动手 | 新项目规划、需求不明确场景 |
| **Pipeline** | 多步严格顺序，明确输入/输出/通过条件 | 多阶段内容生产 |

推荐组合：Pipeline + Reviewer（多阶段生成+审查）、Generator + Inversion（采访后生成）。^[raw/articles/agent-skill-spec-building-design-patterns.md]

## 第 2 来源 — Agent Plugins 1.0.0：跨客户端打包标准（Vercel，2026-08-06）

Vercel 发布的开放、厂商中立标准，为 Skill + MCP server 提供统一的**可分发打包格式**——解决同一组件在不同客户端格式下需重复适配的问题。^[raw/articles/vercel-agent-plugins-skill-mcp-packaging-2026.md]

### 互补角度（5 条）

1. **目录契约**：`plugin.json`（最低要求 `$schema` + `name`）+ 固定目录结构——`skills/` 放 SKILL.md（含 scripts/references）、`mcp.json` 放 MCP 配置、`com.example.client/` 放客户端专属扩展^[raw/articles/vercel-agent-plugins-skill-mcp-packaging-2026.md]
2. **组件独立校验**：客户端校验 manifest 后组件各自独立验证——一个组件无效不拖垮其他组件^[raw/articles/vercel-agent-plugins-skill-mcp-packaging-2026.md]
3. **不做重定义**：Skill 与 MCP 已有各自规范，Agent Plugins 只定义"客户端如何一起发现组件"，不重新定义组件本身（commands/hooks/agents 留给客户端）^[raw/articles/vercel-agent-plugins-skill-mcp-packaging-2026.md]
- **小而有意**：v1 只聚焦两种组件类型（Agent Skills + MCP servers），其余由 Technical Steering Committee 后续扩展^[raw/articles/vercel-agent-plugins-skill-mcp-packaging-2026.md]
- **与 SKILL.md 标准的关系**：本文档 21-25 行的 Anthropic SKILL.md 格式（scripts/references/assets）与 Agent Plugins 的 `skills/summarize/{SKILL.md,scripts,references}` 目录结构直接兼容——打包标准复用而非替换 Skill 规范^[raw/articles/vercel-agent-plugins-skill-mcp-packaging-2026.md]

→ [[raw/articles/vercel-agent-plugins-skill-mcp-packaging-2026|第 2 来源原文存档]]

→ [[raw/articles/agent-skill-spec-building-design-patterns|原文存档]]

---
## 深度分析

把两个来源放在一起看，会得到单篇阅读时看不到的三层结构判断。

**第一层：规范层与打包层的分工正在收敛。** Anthropic 的 SKILL.md 标准回答"一个 Skill 长什么样"，Agent Plugins 1.0.0 回答"多个组件如何一起分发"。两者刻意不重叠：plugin.json 最低只要 `$schema` + `name` 两个字段，其余契约交给目录结构本身——这正是 Unix "约定优于配置"哲学在 Agent 生态的复刻。Skill 的 `skills/summarize/{SKILL.md,scripts,references}` 子目录在 plugin 内原样保留，说明打包标准复用而非替换组件规范^[raw/articles/agent-skill-spec-building-design-patterns.md, raw/articles/vercel-agent-plugins-skill-mcp-packaging-2026.md]。

**第二层：加载经济学决定了一切设计。** 三层渐进式加载把 Token 成本从"每个 Skill 全文常驻"压到"目录级 ~50-100 tokens 常驻"，20 个 Skill 初始加载仅 1000-2000 tokens。这不是实现细节，而是整个设计空间的约束函数：Description 只写触发条件不写工作流程（否则 Agent 按 description 执行、跳过正文）、Tool Wrapper 模式把完整规范推到 references/ 按需读、Agent Plugins 把重活留给客户端——全都是同一个约束的不同投影。理解了 L1/L2/L3 的成本结构，五个设计模式的选择标准就自然浮现。

**第三层：评估链是把 Skill 当软件工程做的关键。** Skill-Creator 的 Grader→Comparator→Analyzer 三 Agent 链，本质是把 ML 的"泛化而非过拟合"纪律迁移到 Prompt Engineering：不为测试用例针对性改 Skill、双盲比较、事后揭盲归因。这与 [[entities/agent-skill-writing-advanced|Skill 高级写作]]、[[entities/agent-skill-writing-evaluation|Skill 评估]] 形成方法闭环——写作产出 Skill，评估链决定 Skill 是否值得保留。

跨来源的张力点也值得记录：Anthropic 路线假设单客户端内的最佳实践（Skill-Creator、评估链都是客户端内工具），而 Agent Plugins 路线假设组件要在五个以上客户端间流转（ChatGPT/Codex、Cursor、Copilot、Kiro、VS Code）。前者优化深度，后者优化可移植性；v1 明确"小而有意"，把 commands/hooks/agents 留在客户端侧，就是在两种张力之间划的最小可行边界^[raw/articles/vercel-agent-plugins-skill-mcp-packaging-2026.md]。

## 实践启示

- **写 Description 前先问"什么任务该触发它"**：只描述触发条件，一个字都不要写工作流程。这是整个规范里性价比最高的一条纪律——写错它，L2 正文永远不会被读到。
- **Token 预算当作硬约束来设计**：L1 控制在 ~100 tokens 内，L2 <5000 tokens，L3 承担体积。Skill 超重时先想"哪些内容可以降到 references/"，而不是压缩措辞——这正是 Tool Wrapper 模式的适用信号。
- **选择设计模式看两件事：任务是否有固定阶段、用户是否知道要什么。** 有阶段用 Pipeline（+Reviewer 做阶段间质量闸门）；用户说不清需求用 Inversion 先采访；两者都不明显再考虑 Generator。单一模式够用就不要组合。
- **Skill 出现"第三次重复"时立即抽取**：Agent 反复写同类辅助脚本，就把脚本落进 `scripts/` 直接调用——重复模式是Skill 陷入过拟合的最早信号。
- **要跨客户端分发的组件，从第一天就按 Agent Plugins 目录布局组织**：`plugin.json` + `skills/` + `mcp.json` 的结构成本几乎为零，而事后重打包的成本极高。纯客户端内使用的 Skill 则不必为此付出结构税。
- **评估不是可选项**：修改 Skill 后至少跑一轮 Grader 评分 + Comparator 盲比，避免"为单个坏 case 改一次就回归一次"的过拟合循环。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

