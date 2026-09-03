---
title: Hermes Agent 新手上手指南
created: 2026-05-07
updated: 2026-08-01
type: concept
tags: [hermes, agent, tutorial, workflow]
related:
  - [[raw/articles/hermes-agent-newbie-guide-dotta|原文存档]]
  - [[entities/skill-development-guide-aliyun-2026|重新定义Skill开发：保姆级教程&一站式开发助手发布]]
  - [[entities/qoder-skills-完全指南从零开始让-ai-按你的标准执行-v2|Qoder Skills 完全指南：从零开始，让 AI 按你的标准执行]]
sources: ['raw/articles/hermes-agent-newbie-guide-dotta']
review_value: 7
review_confidence: 8
review_recommendation: worth-reading
review_stars: 3
confidence: high
---
# Hermes Agent 新手上手指南

## 核心定位
> **模型是大脑，Hermes 是工作台。**
- Claude / GPT / DeepSeek 负责推理和生成
- Hermes 负责把模型接进长期运行的 Agent 系统：记忆沉淀、Skill 积累、消息平台接入、定时任务、MCP 工具

## 适合人群
| 适合 | 不适合 |
|------|--------|
| 想让 AI 长期处理资料、代码、自动化任务 | 只想偶尔问答 |
| 已在用 Claude/GPT 但想要本地 Agent 工作流 | 不愿意碰命令行 |
| 用过 OpenClaw 想尝试更强记忆和 Skill 体系 | 对稳定性要求极高且没时间排查 |
| 愿意折腾命令行、配置和开源工具 | 企业有严格审计要求且未做隔离 |

## 新手四步
1. **找对资料** — 官方文档优先（hermes-agent.nousresearch.com/docs）、GitHub、CLI Reference；中文社区（hermesagent.org.cn）降门槛用
2. **掌握命令** — `hermes setup` / `hermes doctor` / `hermes status` / `hermes gateway` / `hermes sessions list` / `hermes insights`
3. **写好规则** — 先写 SOUL.md（实例级协作风格）+ AGENTS.md（项目级工作规则），再装 Skill
4. **按需补工具** — Repomix → Tokscale → Hindsight → WebUI → Mission Control

## 关键命令参考
```bash
# 启动和对话
hermes                           # 交互式 CLI
hermes --continue                 # 继续最近一次会话
hermes chat -q "问题"             # 一次性问答
# 配置
hermes setup                      # 首次配置
hermes model                      # 切换模型
hermes config show / edit         # 查看/编辑配置
# 排查
hermes doctor                     # 诊断问题
hermes status                     # 查看状态
hermes logs                       # 查看日志
# 消息网关
hermes gateway setup/run/status   # 接入 Telegram/Discord/飞书/微信
# 会话与成本
hermes sessions list              # 历史会话
hermes insights                   # Token 和成本
# 从 OpenClaw 迁移
hermes claw migrate --dry-run     # 先 dry-run 看迁移内容
```

## SOUL.md vs AGENTS.md
| 文件 | 作用 | 路径 |
|------|------|------|
| `SOUL.md` | 实例级：默认语气、协作方式、行为风格 | `[本地运行时路径已隐藏]` |
| `AGENTS.md` | 项目级：目录结构、测试命令、代码风格、禁止事项 | 项目根目录 |

## 上手指南节奏
**第 1 天**：跑通基础（`hermes setup` → `hermes model` → 第一次对话）
**第 2-3 天**：熟悉高频命令（doctor / status / gateway / sessions / insights）
**第 1 周**：写 SOUL.md + 常用项目 AGENTS.md，观察常误解点并补规则
**第 2 周以后**：按需上 Repomix → Tokscale → Hindsight → WebUI

## 不变原则
1. 先写规则，再做任务
2. 先建立记忆，再追自动化
3. 先看官方文档，再看社区教程
4. 先把它用稳，再谈进阶

---

## Hermes 自进化机制：Memory、Skill 与 Session Search 三层闭环

Hermes 与传统 Agent 脚手架最核心的差异在于其**自进化（Self-Improve）机制**。不同于 Claude Code 等工具依赖人工事先定义好的规则体系，Hermes 设计了一套让 Agent 能够**主动沉淀经验、自主维护技能**的完整闭环。^[raw/articles/llm-agent脚手架如何具备自进化能力以hermes-agent为例.md]

这一机制基于三层记忆架构的协同：

| 机制 | 存储内容 | 使用方式 | 自进化中的角色 |
|------|---------|---------|--------------|
| **Memory** | 稳定事实（用户偏好、环境信息） | 每次会话自动注入 System Prompt；通过 `memory` 工具主动添加/修改 | 避免重复询问已确认的信息 |
| **Skill** | 程序性知识（操作步骤、最佳实践） | System Prompt 中显示索引（名称+描述），通过 `skill_view` 按需加载完整内容；通过 `skill_manage` 创建/修改/删除 | 避免重复发明已验证的流程 |
| **Session Search** | 原始对话轨迹（试错过程、顿悟时刻） | 通过 `session_search` 工具主动搜索关键词，LLM 生成结构化摘要 | 避免重复犯错已踩过的坑 |

三层之间的协作逻辑是：**Memory 管"事实"，Skill 管"方法"，Session Search 管"过程"**。这一设计避免了记忆混淆——模型不会把用户偏好当成操作流程，也不会把试错过程当作稳定规范。^[raw/articles/llm-agent脚手架如何具备自进化能力以hermes-agent为例.md]

一个典型的自进化闭环如下：
1. 用户问"帮我把这周的 HN 头条整理成摘要发到 Telegram"
2. Agent 启动，System Prompt 自动包含 USER.md（"User prefers short summaries"）和 Skills 索引（发现 "weekly-digest" 技能相关）
3. Agent 调用 `skill_view("weekly-digest")` 加载技能，按步骤执行
4. 执行中发现技能写的"RSS 抓取"过时了，用户实际用 HN API。Agent 调用 `skill_manage(action='patch')` 更新技能
5. 用户说"以后只要 5 条头条"。Agent 调用 `memory(action='add', target='user', content='User wants HN digest capped at 5 headlines')`
6. 下周用户再问同样的问题——启动时 System Prompt 已经包含新记忆和修补后的技能

**这个循环的关键是：每一次任务的起点都比上一次更高。**

### Skill 工具化的设计哲学

Hermes 为 Skill 系统专门设计了 `skill_manage` 这个多功能合一工具（create/patch/edit/delete/write_file/remove_file 六种能力），而非仅仅提供基础的 read/write 文件操作。这一设计遵循了 **"高频专用、低频通用"原则**：当一个操作在 Agent 生命周期内会被频繁调用时，专用工具能显著降低模型调用难度，减少构造错误命令的概率。^[raw/articles/llm-agent脚手架如何具备自进化能力以hermes-agent为例.md]

更关键的是，Hermes 的 Skill 系统是**让 Agent 自主维护**的——不是人提前定义好让模型照搬，而是模型在完成任务过程中主动创建、发现过时时立即 patch。这意味着 Skill 系统必须足够易用，Agent 才愿意频繁操作它。因此 `skill_manage` 6 种能力统一封装、减少工具定义个数的做法，本质上是降低 Agent 的认知负担。

### Memory 写入的 Prompt 注入防护

由于 Memory 内容会在下次会话时自动注入 System Prompt，Hermes 在 Memory 写入前嵌入了 `_MEMORY_THREAT_PATTERNS` 正则扫描，防止攻击者通过诱导 Agent 记录恶意内容来**持久化污染 System Prompt**——每次新会话都会携带污染内容。防护策略采用正则匹配而非 LLM 判断，优势在于零 LLM 调用成本、确定性高、不存在误判风险。^[raw/articles/llm-agent脚手架如何具备自进化能力以hermes-agent为例.md]

这是一个容易被忽视但至关重要的设计细节：凡是会进入 System Prompt 或影响 Agent 行为的持久化存储（Memory、Skill、甚至工具描述本身），在写入前必须经过严格的输入校验。

---

## 消息网关与多平台接入：从 Telegram 到飞书的配置实践

Hermes 的 `hermes gateway` 命令是其区别于纯 CLI Agent 的核心差异化能力之一。通过 Gateway，Hermes 可以接入 Telegram、Discord、飞书、微信等消息平台，将 Agent 能力延伸到日常沟通工具中。

[[entities/tencent-vibe-coding-to-agentic-engineering-backend|腾讯 Agentic Engineering 实践]] 中提到的 Skill/Command/MCP 三层架构为理解 Gateway 的定位提供了参照：MCP（Model Context Protocol）Server 是**外部系统的适配层**，公开协议标准使得接入外部系统不需要修改 Agent 核心代码。Hermes Gateway 的设计与此一脉相承——它扮演的是消息平台的 MCP Server 角色，把来自不同平台的输入格式统一转换为 Agent 可处理的结构化消息。

Gateway 配置的核心挑战不在连接本身，而在于**消息路由与会话管理**。当用户通过 Telegram 发起一个请求，Agent 处理后通过 Discord 回复另一个用户时，两个会话的上下文必须严格隔离——这要求 Gateway 层维护独立的会话状态，而非共享同一个 System Prompt。Hermes 的会话管理设计（`hermes sessions list` 查看历史会话）支持这一点，但需要用户在配置时显式指定每个平台对应的会话隔离策略。

对于多平台用户，一个常见的配置模式是：
- **Telegram/Discord**：适合异步任务（"帮我整理这篇文档"）
- **飞书/微信**：适合即时查询（"这个错误怎么解决"）

两者的区别在于：异步任务需要完整的上下文（完整的 Skill 加载、Memory 注入），而即时查询更依赖单轮的快速响应。Gateway 配置时应根据平台特性选择不同的默认启动参数。

---

## 相关页面
- [[concepts/hermes-agent]] — Hermes 概述
- [[concepts/hermes-agent]] — 自进化机制
- [[concepts/hermes-agent-skill]] — Skill 系统
- [[raw/articles/hermes-agent-newbie-guide-dotta|原文存档]]
## 关联实体

**上游依赖**:
- [[entities/skill-development-guide-aliyun-2026]] — 提供基础理论/方法
- [[entities/qoder-skills-完全指南从零开始让-ai-按你的标准执行-v2]] — 提供基础理论/方法
- [[entities/tencent-vibe-coding-to-agentic-engineering-backend]] — 提供基础理论/方法

**下游应用**:
- [[entities/ai-employment-eight-changes-tencent-research]] — 具体应用场景
- [[entities/从-anthropic-到-googleagent-skills-正在进入设计模式阶段]] — 具体应用场景
- [[entities/yumanju-ai-full-flow-efficiency]] — 具体应用场景

**平行协作**:
- [[entities/llm-agent脚手架如何具备自进化能力以hermes-agent为例]] — 替代/补充方案
- [[entities/tencent-vibe-coding-to-agentic-engineering-backend]] — 替代/补充方案
- [[entities/10x-is-a-lot]] — 替代/补充方案


→ [[raw/articles/hermes-agent-newbie-guide-dotta|原文存档]]

## 相关实体
- [[entities/skill-development-guide-aliyun-2026|重新定义Skill开发：保姆级教程&一站式开发助手发布]]
- [[entities/qoder-skills-完全指南从零开始让-ai-按你的标准执行-v2|Qoder Skills 完全指南：从零开始，让 AI 按你的标准执行]]
- [[entities/ai-employment-eight-changes-tencent-research|AI 行业就业八大变化（腾讯研究院纵向对比）]]
- [[entities/从-anthropic-到-googleagent-skills-正在进入设计模式阶段|Agent Skill 设计模式]]
- [[entities/yumanju-ai-full-flow-efficiency|柚漫剧 AI 全流程提效拆解]]
- [[entities/cdp-bridge-mcp-real-browser-agent|CDP Bridge MCP：真实浏览器直连 MCP 工具]]
- [[entities/four-sub-agent-patterns|四种 Sub Agent 模式]]
- [[entities/llm-agent脚手架如何具备自进化能力以hermes-agent为例|Hermes Agent 自进化机制详解]] — Memory/Skill/Session Search 三层闭环
- [[entities/tencent-vibe-coding-to-agentic-engineering-backend|腾讯 Agentic Engineering 实践]] — Skill/Command/MCP 三层架构

## 新增关联实体
- [[entities/10x-is-a-lot]]
- [[entities/how-developers-can-build-agentic-agreement-workflows-on-docu]]
- [[entities/不用再学ai了生成结果包稳的agent来了]]
- [[entities/还在手写-osgetenvpydantic-settings-让你配置管理效率翻倍]]

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
