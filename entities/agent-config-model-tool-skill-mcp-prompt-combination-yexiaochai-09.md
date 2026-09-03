---
title: "第 09 篇 · Agent 配置：模型、工具、技能、MCP 与提示词的组合"
created: 2026-07-12
updated: 2026-08-01
type: entity
tags: [agent, harness, configuration, skill, mcp, tool, prompt, harness-engineering]
confidence: 0.75
provenance_state: extracted
sources:
  - raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09
---

# 第 09 篇 · Agent 配置：模型、工具、技能、MCP 与提示词的组合

> Author: 叶小钗 | Source: 微信公众号

本文是叶小钗 Agent 系列教程的第 09 篇，聚焦于 Agent 配置层的设计与实现 —— 将 Agent 的模型选择、工具、技能、MCP 和系统提示词从代码中解耦为外部配置，实现**"一套代码，多 Agent 组装"**的能力。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

## 代码 vs 配置的分界线

叶小钗提出明确的分离原则：^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md] 不是所有内容都适合抽取为配置 —— 需要区分系统机制与业务策略。

**代码实现（系统机制，不因用户改变）**：^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

- 工具的具体实现（Python 函数）
- 技能的执行引擎（加载、解析、执行流程）
- Agent Loop 循环机制（消息调度、状态管理）
- SSE 协议（通信层）
- MCP 协议处理

**配置部分（产生不同的业务 Agent）**：^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

- 模型的选择（LLM 后端、参数配置）
- Agent 需要使用的工具列表
- Agent 需要使用的技能列表
- MCP 服务器配置
- Agent 的系统提示词（长文本行为定义）

这些配置的不同组合可以组装出不同的业务 Agent。同一套代码基，差异化的就是这几样配置项，形成**数据驱动行为**的架构模式。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

## 配置设计方案

### 为什么用 JSON 文件而非数据库

对于个人 / 团队级 Agent 工具，JSON 文件配置相比传统数据库方案有显著优势：^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

- **可读可手编**：直接打开 JSON 文件即可查看所有 Agent 配置，无需额外 UI 开发
- **可版本化**：配置文件纳入 Git 管理，任何变更通过 `git diff` 一目了然
- **简化部署**：无数据库依赖，Agent 分享给其他人无需同步数据库 Schema
- **无并发问题**：个人 Agent 通常单机单用户使用，无并发写入竞争

### 配置的物理结构

配置采用的是**文件 + 目录**的组合方案：^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

```
workspace/
├── models.json          # 模型配置
├── tools.json           # 工具配置
├── skills/              # 技能目录（每个技能一个子目录）
├── agents.json          # Agent 索引（结构化字段）
├── agents/              # Agent 提示词目录
│   └── 00000000/
│       └── agent.md     # Agent 长文本系统提示词
└── mcp_servers.json     # MCP 服务器配置
```

这种分离设计是有意为之：^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]
- `agents.json` 保存结构化字段（模型 ID、工具 ID 列表、技能 ID），适合校验和筛选
- `agent.md` 保存长文本系统提示词，适合用 Markdown 编辑和版本管理

### Agent Schema 设计

核心 Schema 将 Agent 定义为一个**关联聚合**，不直接内联工具定义或模型密钥：^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

```
Agent
├── model_id → models.json 中的一条模型配置
├── tool_ids → tools.json 中的配置 ID 列表
├── skill_ids → skills.json 中的配置 ID 列表
├── mcp_server_ids → mcp_servers.json 中的配置
└── agent.md → 该 Agent 自己的角色与行为要求的提示词
```

这种通过 ID 引用的松耦合设计，使得 Agent 可以灵活组合各项能力而不需要重复存储基础配置。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]


## 与 Harness Engineering 的关系

本文的"配置驱动 Agent"思想与 Harness Engineering 中 Context Management 和 Working Set 概念一致 —— 关注的是 Agent 的能力边界定义与运行时编排，而非底层实现的细节。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

具体映射关系：
- Agent 的模型/工具/技能选择 → **Working Set 定义**（Agent 在当前任务中可用的资源集合）
- Agent 配置的 JSON 存储 → **Skill 的外部化配置**（将技能参数从代码解耦）
- Agent.md 长提示词 → **Context Template**（固定化的行为规范注入）

## 深度分析

### 配置驱动 vs 代码驱动的分界线哲学

本文最重要的贡献不是技术实现，而是提出了 Agent 系统中"什么应该写死、什么应该可配"的分界原则。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md] 这一原则在软件工程中有长期实践的：Unix 哲学（机制与策略分离）、IoC 容器（控制反转）、微服务配置中心。Agent 工程正在经历同样的架构演进 —— 从 monolith Agent（一个 Agent 做所有事）进化为组合式 Agent 平台（配置驱动多 Agent 组装）。

分界线判断标准：**是否因用户/场景变化而变化**。工具的实现逻辑不因用户变化（所有用户都用同一个 Python `read_file` 函数），但模型选择、技能组合因场景不同而异。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

### JSON 文件 vs 数据库：Agent 配置存储的取舍

选择 JSON 文件而非数据库反映了个人 Agent 与生产级 Agent 的核心差异：^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

| 维度 | JSON 文件 | 数据库 |
|------|-----------|--------|
| **可读性** | 直接编辑，零学习成本 | 需管理工具 |
| **版本管理** | 天然 Git 友好 | 需额外工具 |
| **并发控制** | 无（单用户场景） | ACID 事务 |
| **扩展性** | 不适用于多用户 | 高并发支撑 |
| **部署复杂度** | 低（启动即用） | 高（需维护 DB） |

选择 JSON 的关键前提是**配置文件无并发写入问题**。当 Agent 从个人工具演变为多人协作平台时，数据库迁移是必要的架构升级。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

### ID 引用式 Schema：松耦合的 Agent 组件模型

通过 ID 引用而非内联存储的 Schema 设计，体现了**组件化 Agent 构建**的思想。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md] 每个 Agent 是一个"组件装配图"而不是"自包含单体"：

- **复用性**：同一模型配置可在多个 Agent 间共享，一处修改全局生效
- **可组合性**：工具、技能、MCP 服务器可自由组合成不同 Agent
- **可观测性**：通过查看 Agent 引用的 ID 即可了解其能力边界

这与当代前端框架（React 组件组合）和后端微服务（服务编排）的设计理念一致 —— Agent 组件化是 Harness Engineering 的自然演进方向。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

### 长文本提示词与结构化配置的分离设计

将 Agent 配置拆分为 `agents.json`（结构化字段）和 `agent.md`（长文本提示词）展示了一重要的设计模式：**结构化数据的校验友好性与自由文本的表达力之间的平衡**。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

- 结构化数据（JSON）适合：ID 关联、字段校验、列表筛选、自动补全
- 自由文本（Markdown）适合：角色定义、行为规范、复杂约束、Few-Shot 示例

这种分离类似于编程语言中类型声明与实现体分离 —— Schema 定义"有什么"，`agent.md` 定义"怎么用"。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]


## 实践启示

1. **尽早建立代码/配置分界线**：设计 Agent 系统时，预先定义"哪些是可变的业务策略、哪些是稳定的系统机制"。可变部分设计为外部配置，减少未来改动代码的需求。分界线判断标准：是否因用户/场景变化而变化。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

2. **配置版本化是必选项**：将 Agent 配置纳入版本管理（Git），确保配置变更可追溯、可回滚。即使使用数据库存储，也应保持配置变更的审计日志。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

3. **组件化 Agent 构建**：通过 ID 引用而非内联存储组装 Agent。模型、工具、技能、MCP 作为独立组件管理，Agent 是这些组件的装配体。设计时关注组件间的接口兼容性。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

4. **结构化 + 自由文本混合配置**：将 Agent 的关键配置拆分为结构化部分（校验友好、适合筛选）和自由文本部分（表达力强、适合复杂约束）。不要把长文本提示词塞进 JSON 字段，也不要把结构化数据硬编码在 Markdown 中。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

5. **从单 Agent 到多 Agent 平台的演进路径**：先以 JSON 文件快速启动个人 Agent 工具，当 Agent 数量增长到需要多用户协作时，平滑迁移到数据库存储。保持配置 Schema 不变，只换存储层。^[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09.md]

## 相关实体

- [[entities/harness-engineering-core-patterns-claude-code|Harness 工程核心模式]]
- [[entities/lilian-weng-harness-engineering-self-improvement|Harness Engineering]]
- [[entities/anthropic-mcp-revisited-tool-search-code-orchestration|MCP 协议与工具编排]]
- [[entities/anthropic-14-skill-patterns-best-practices|Skill 驱动开发]]
- [[entities/agent-harness-context-management-working-set|Working Set 上下文管理]]

→ [[raw/articles/agent-config-model-tool-skill-mcp-prompt-combination-yexiaochai-09|原文存档]]
