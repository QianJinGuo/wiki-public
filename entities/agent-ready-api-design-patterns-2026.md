---
title: "面向 Agent 的 API 设计（Agent-Ready API）"
created: 2026-09-05
updated: 2026-09-12
type: entity
tags: [api-design, agent, mcp, agent-integration, tool-selection]
sources: [raw/articles/agent-ready-api-design-restless-2026]
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
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

## 深度分析

### 响应即接口：通信信道只剩一条

Koberger 的论证起点是一个被低估的观察：agent 不做研究，它调用并反应。人类开发者撞上 400 会去读文档、搜 Stack Overflow、翻源码；agent 则会重试、猜测近似约定、依赖参数化知识（parametric knowledge），却极少主动检索文档。这意味着响应体从「状态描述」升级为唯一可靠的通信信道——凡是修复所需的信息必须随响应一起抵达，否则对 agent 而言等同于不存在。Restless 的 Agent Recovery 把这一点工程化：请求按 error、endpoint、parameters 做指纹（fingerprint），下一步动作（recovery 文本、log 链接、CLI 复现命令）作为 continuation 直接注入 error JSON。更细腻的一层是：agent 对带跟踪参数或 query params 的外链天然不信任，只有链接出现在散文（prose）中并附上明确理由时，它才更愿意跟随。于是设计者必须区分两类信息——机器可执行的指令要内联，需要 agent 主动跳转的链接则必须附加动机。^[raw/articles/agent-ready-api-design-restless-2026.md:22-73]

### 30-50 工具退化：这是可用性带宽，不是性能瓶颈

「MCP 工具选择在约 30-50 个工具后退化」是全篇最具迁移价值的硬事实。它并非推理延迟问题，而是选择质量问题：候选工具一多，agent 的判选精度下降，越界调用与漏调随之上升。这相当于给 agent 系统划出一条**可用性带宽上限**——工具面的规模本身就是设计变量，而不是无成本的资源。三条对应解法各有适用面：以用量与信号（`agent_send_feedback`、最常搜索的工具）自动精选短列表；用 Tool Search 保留长尾的可发现性；把若干端点合成为单个语义单元（Restless 称为 Usecases）。三者组合，本质是把「检索—选择」的负担从 agent 的上下文里挪回服务端。该结论与 [[concepts/lost-in-the-middle|Lost in the Middle]] 描述的位置敏感退化同源，也与 [[entities/mcp-tool-design-tradeoffs-anthropic-2026|MCP 工具设计权衡]] 的取舍框架互补。^[raw/articles/agent-ready-api-design-restless-2026.md:98-116]

### 身份与信任的位移

集成链条最脆弱的一环不在技术调用，而在账号创建：多数应用要求人类去网站注册、再把工具配好，agent 才能继续。`auth.md` 用「向用户邮箱发送 token 完成验证」把这一步程序化，给 agent 受作用域限定（scoped）的凭据自行推进；终局则是 agent provider（Claude、OpenAI 等）直接为用户身份背书，从而跳过该步。其意义超出便利性——它把「谁被授权、授权到什么范围」从会话级的人类登录，重构为可委派、可细粒度的凭据面，与 [[concepts/agent-identity-portability|Agent 身份可移植性]] 的议题直接接续。^[raw/articles/agent-ready-api-design-restless-2026.md:117-130]

### 双流设计：人类与 agent 是两类一等用户

CLI 是最好的反例样本。好的 CLI onboarding 是交互式的，而 agent 处理不了 TTY——于是正确的做法不是把交互简化掉，而是为同一工具构建两条并行流：人类一条、agent 一条。检测手段已经标准化：agent provider 注入环境变量（Claude Code 的 `CLAUDECODE=1`、Codex 的 `CODEX_SANDBOX`），语言层则暴露 `process.stdout.isTTY`（Node）或 `sys.stdout.isatty()`（Python）作为兜底。更关键的是，识别出 agent 之后不只是「去掉交互」，还应主动提高冗余度（verbose），把它因缺乏上下文而缺失的信息在输出里补齐。这与 [[entities/cli-agent-patterns-mcp-shell-agents|CLI Agent 模式]] 中把 CLI 当作 agent 一等接口的定位一致。^[raw/articles/agent-ready-api-design-restless-2026.md:131-153]

### agent-only 端点的代价与解放

传统上 API 是与开发者的长期契约，稳定性被置于最高价值。Koberger 主张：既然 API 现在是 agent 与产品交互的方式，就可以存在 agent-only 端点，随产品高频迭代——持续打磨 endpoint 的 ergonomics、删除未使用的端点、跟上产品演进。这在换来迭代自由的同时引入新的治理责任：两套消费面意味着两套契约与两套兼容性承诺，团队必须先明确「哪些端点对哪类消费者承诺稳定」，否则「随时可改」会退化为「随时会坏」。^[raw/articles/agent-ready-api-design-restless-2026.md:154-162]

## 实践启示

1. **把错误体当作恢复协议，而非状态报告。** 每条错误至少携带三件东西：怎么修（recovery 指令）、去哪看（log 链接）、怎么复现（CLI 或请求指纹）。对人类都嫌不足的 `invalid_parameter: date_range`，对 agent 等于零信息。^[raw/articles/agent-ready-api-design-restless-2026.md:22-73]
2. **给工具面设硬上限，并按信号自动精简。** 把 30-50 当作默认警戒线：以使用量与反馈信号精选短列表，长尾交给 Tool Search，高频组合合成为 Usecase 级工具。能力扩张时必须同步评估选择精度，而不是只看覆盖度。^[raw/articles/agent-ready-api-design-restless-2026.md:98-116]
3. **开通反馈与日志两条回传通道。** 暴露 `agent_send_feedback` 让 agent 自报卡点并分类（feature request／缺文档／错误难懂）；再把生产请求日志（经 Sentry 等）接给 agent，让它能调试真实数据下的失败——多数 API 问题只在生产环境才暴露。^[raw/articles/agent-ready-api-design-restless-2026.md:74-97]
4. **为每个面向 agent 的 CLI 预留非 TTY 分支。** 用 `CLAUDECODE=1`、`CODEX_SANDBOX`、`isatty()` 检测运行环境并切换到非交互、更高冗余度的输出；必要时在同一命令下并行维护人类流与 agent 流。^[raw/articles/agent-ready-api-design-restless-2026.md:131-153]
5. **把 onboarding 断点程序化。** 账号创建是集成链最易断裂处，优先采用 `auth.md` 式「邮箱 token 验证 + scoped credentials」流程，让 agent 在不请人类出场的情况下拿到最小必要权限。^[raw/articles/agent-ready-api-design-restless-2026.md:117-130]
6. **先划定契约边界，再开放 agent-only 端点。** 迭代自由以治理为前提：明确哪组端点服务 agent、其变更节奏与兼容承诺，避免把「可快速迭代」变成对调用方的隐性破坏。

## 相关

- [[entities/anthropic-mcp-revisited|Anthropic MCP Revisited]]
- [[entities/anthropic-12-mcp-production-patterns|MCP 12 个生产模式]]
- [[entities/agent-eval-counterintuitive-insights-langfuse|Agent Eval 反直觉洞察]]

→ [[raw/articles/agent-ready-api-design-restless-2026|原文存档]]