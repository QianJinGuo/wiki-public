---
title: "Model Context Protocol (MCP)"
created: 2026-05-21
updated: 2026-08-06
type: concept
tags: [mcp, agent, tool, protocol, architecture, integration]
sources:
  - raw/articles/anthropic-12-mcp-production-patterns
  - raw/articles/higress-mcp-stateless-http-paradigm-aliyun-2026-08-06
summary: MCP（Model Context Protocol）是 Anthropic 提出的 Agent 连接外部系统的标准协议。本文综合 12 个生产级 MCP 设计模式，涵盖工具交互面设计、认证架构、上下文经济、和打包分发五大类别。
---

# Model Context Protocol (MCP)

> MCP = Model Context Protocol，不只是协议，而是**面向 Agent 的产品交互面**

MCP 是 Anthropic 提出的用于连接 AI Agent 与外部系统（工具、数据、权限系统）的标准协议。本文综合官方生产级最佳实践，总结 12 个可复用的 MCP 设计模式。

## 一、背景与定位

### 1.1 为什么需要 MCP

生产级 Agent 的难点不是「能不能调用工具」，而是「能不能安全、稳定、低成本地连接真实系统」。

MCP 解决的核心问题：
- **API 差异**：每个外部系统有自己的 API 设计和认证方式
- **工具粒度**：如何将底层 API 封装成 Agent 可用的工具
- **上下文成本**：大量工具定义消耗宝贵的上下文预算
- **认证安全**：凭证管理是生产级集成的门槛

### 1.2 MCP 的三层架构

```
┌─────────────────────────────────────────┐
│         Agent (Claude, etc.)            │
├─────────────────────────────────────────┤
│           MCP Client                     │
├─────────────────────────────────────────┤
│         MCP Server                      │
│  (Remote-First Server Pattern)          │
├─────────────────────────────────────────┤
│     External Systems (API, DB, etc.)    │
└─────────────────────────────────────────┘
```

## 二、五组 12 模式详解

### 第一组：工具交互面（Tool Surface）

#### 模式 1：远程优先服务器模式（Remote-First Server Pattern）

**问题**：MCP Server 应该运行在哪里？

| 形态 | 适用场景 |
|------|---------|
| 本地 MCP Server（stdio） | 桌面应用、IDE Agent、Claude Code、命令行场景 |
| 远程 MCP Server | 生产环境（浏览器/移动端/云端/托管平台） |

**远程优先的好处**：
- 一个 Server 服务多个客户端
- 认证流程跨环境复用
- Web/移动端/云端 Agent 都能访问
- Server 可独立部署、扩展、监控和审计

**判断原则**：本地 MCP Server 适合开发者环境，远程 MCP Server 才是生产分发形态。

#### 模式 2：按意图组织工具模式（Intent-Grouped Tools Pattern）

**问题**：工具应该按什么粒度暴露？

**错误做法**：把每个 API endpoint 1:1 包装成 tool

```python
# 错误：暴露底层 API
get_thread, parse_messages, create_issue, link_attachment
# Agent 需要自己拼接4个底层动作

# 正确：按用户意图封装
create_issue_from_thread  # 内部处理编排、ID归一化、附件关联、错误重试
```

**适合场景**：API 面不算太大、用户任务相对明确的系统（Linear、Slack、Notion、Sentry）

#### 模式 3：薄交互面模式（Thin Surface Pattern）

**问题**：底层 API 面太大（几百~几千个操作），按意图封装也失控，怎么办？

**思路**：不要暴露很多工具，只暴露**少量高能力工具**

典型组合：
- `search`：让 Agent 搜索可用 API 或能力
- `execute`：让 Agent 写短脚本，由服务端在沙箱里执行

**典型案例**：Cloudflare MCP Server，2 个工具覆盖约 2500 个 endpoint，工具定义只需约 1000 tokens。

### 第二组：交互语义（Interaction Semantics）

#### 模式 4：内联 UI 模式（Inline UI Pattern）

**问题**：有些结果不应该被模型描述，而应该直接被用户看到。

**典型场景**：
- 监控 Dashboard、趋势图、搜索结果
- 审批表单、文件预览、状态面板

MCP Apps 允许工具返回一个**可交互界面**，客户端在聊天界面中直接渲染。

#### 模式 5：引导式输入模式（Elicited Input Pattern）

**问题**：Agent 缺少结构化信息时（region、环境、项目ID等），如何处理？

**三个选项**：
1. 猜 → 有风险
2. 回到对话追问 → 打断流程
3. 让工具调用暂停，向用户请求结构化输入 → MCP Form Mode Elicitation

**适用场景**：
- 缺少结构化参数
- 多个候选项需要用户选择
- 删除/支付/部署等高风险操作需要确认
- Server 明确知道还缺什么信息

#### 模式 6：外部跳转交接模式（External Handoff Pattern）

**问题**：有些信息根本不应该经过 MCP Client（OAuth、支付、银行卡、敏感凭证），怎么办？

**MCP URL Mode Elicitation**：Server 返回一个 URL，Client 打开浏览器或外部页面，用户在那里完成 OAuth、支付或敏感信息输入，流程结束后再回到 Server 继续执行。

**区分**：
- **Form Mode**：Server 可以合法接收和处理的结构化输入
- **URL Mode**：应该由第三方或外部系统处理的敏感流程

### 第三组：认证与凭证流（Auth and Credential Flow）

#### 模式 7：可发现认证模式（Discoverable Auth Pattern）

**问题**：每个 MCP Server 都自己发明认证方式 → 接入成本高，客户端难以统一支持。

**CIMD（Client ID Metadata Documents）**：客户端可以通过标准元数据**发现认证方式**。客户端不需要猜这个 Server 怎么登录，而是读取 metadata，按标准流程启动 OAuth。

#### 模式 8：凭证托管到 Vault 模式（Vault-Held Credentials Pattern）

**问题**：token 放在工具参数、环境变量、临时配置里 → 生产系统危险。

**方案**：把凭证生命周期上移到**平台层**。

在 Claude Managed Agents 里，MCP OAuth credential 可以注册到 Vault。创建 session 时引用 vault ID，平台负责把合适的凭证注入到 MCP 连接中，并处理刷新、撤销和生命周期管理。

**关键洞察**：MCP Server 不需要在每次工具调用里接收 token，也不需要自己实现完整的刷新、撤销和轮换逻辑。凭证管理从工具调用路径里抽离出来，变成平台能力。

### 第四组：上下文经济（Context Economy）

#### 模式 9：按需加载工具模式（On-Demand Tool Loading Pattern）

**问题**：连接多个 MCP Server → 可能拥有几百个工具。全量加载 tool definitions = 在任务还没开始前就消耗大量上下文预算。

**Tool Search 的做法**：**延迟加载**
- Agent 先通过搜索工具查找可能相关的工具
- 只把命中的工具定义加载进上下文
- 其余工具保持不可见

**Anthropic 官方测试**：Tool Search 可以让 tool-definition tokens 减少 **85% 以上**，同时保持较高的工具选择准确率。

> "Tool Search 不是替代工具设计，而是倒逼工具描述更像产品文案：准确、可检索、可区分。"

#### 模式 10：程序化工具调用模式（Programmatic Tool Calling Pattern）

**问题**：工具返回的结果（大段JSON/几千行日志/trace树/多页搜索结果）不适合直接给模型看。

**方案**：在代码执行沙箱里处理工具结果：
- 循环调用工具
- 过滤数据、聚合字段、做中间计算
- 只把必要结果放进模型上下文

**Anthropic 官方测试**：这种方式在复杂多步流程里可以**减少约 37% 的 token 使用**。

### 第五组：打包与分发（Packaging and Distribution）

#### 模式 11：插件打包模式（Plugin Bundle Pattern）

**问题**：一个有用的 Agent 集成通常不只是一个 MCP Server，还包括 Skills、hooks、subagents、LSP server、项目约定和工作流说明。这些组件分散安装、分散升级 → 配置漂移和版本错配。

**Claude Code Plugins**：把 Skills、MCP servers、hooks、LSP servers、specialized subagents **统一放进一个插件分发**。

**典型案例**：Cowork data plugin，包含 10 个 Skills 和 8 个 MCP servers，连接 Snowflake、Databricks、BigQuery、Hex 等数据工具。

#### 模式 12：服务器分发 Skills 模式（Server-Distributed Skills Pattern）

**问题**：Agent 有了工具访问权，是否就真的会用？答案通常是否定的。

**Server-Distributed Skills 方向**：由 MCP Server 直接分发与自身能力匹配的 Skills。客户端连接 Server 时，不只获得工具，也获得使用这些工具的 playbook。

**趋势**：未来的 MCP Server 不只分发能力，还会分发使用能力的方法。Agent 集成的竞争点会从「谁有工具」变成「谁能把工具、流程、经验和安全边界一起交付」。

## 三、模式之间的关联

这 12 个模式构成一条完整的 Agent-to-System 连接链：

| 分组 | 解决的问题 |
|------|---------|
| 第一组（工具交互面） | 连接形态问题 |
| 第二组（交互语义） | 用户体验问题 |
| 第三组（认证与凭证） | 安全边界问题 |
| 第四组（上下文经济） | 成本效率问题 |
| 第五组（打包分发） | 交付形态问题 |

## 四、实践启示

1. **从模式 1 开始，先想清楚分发形态**：本地转远程的迁移成本非常高，在 API 设计阶段就要规划好
2. **工具粒度决策先用「意图探测」方法**：让 Agent 看工具列表，看它能否在 5 分钟内说清楚能用这些工具完成什么任务
3. **Form Mode 设计优先考虑「不打断」体验**：预判 Agent 可能缺什么信息，在工具描述里提前要求必填
4. **凭证托管是生产级集成的门槛**：凭证安全是生产级 Agent 集成的底线
5. **工具描述要同时满足人和机器**：工具描述应该包含功能名称、输入参数说明、输出结果说明、适用场景、不适用场景

## 相关实体

- [[entities/anthropic-12-mcp-production-patterns]]
- [[entities/amazon-bedrock-agentcore-gateway-mcp-extension]]
- [[entities/agent-harness-architecture-deep-dive-aksahy]]
- [[entities/agent-security-three-step-sequence-harness-governance-identity-crewai]]

## 五、关联实体

- [[entities/anthropic-12-mcp-production-patterns]] — 12 个 MCP 生产模式原文
- [[entities/agent-workflows]] — Agent 工作流与 MCP 集成
- Kiro Graviton（已删除） — Kiro MCP Server 案例
- [[concepts/harness-engineering-framework]] — Harness 工程框架
- [[concepts/open-source-ai-ecosystem|Open Source AI Ecosystem]] — MCP 工具生态是开源 AI 生态系统的重要赛道

## 关联实体

**上游依赖**:
- [[entities/anthropic-12-mcp-production-patterns]] — 提供基础理论/方法
- [[entities/amazon-bedrock-agentcore-gateway-mcp-extension]] — 提供基础理论/方法

**下游应用**:
- [[entities/agent-harness-architecture-deep-dive-aksahy]] — 具体应用场景
- [[entities/agent-security-three-step-sequence-harness-governance-identity-crewai]] — 具体应用场景

**平行协作**:
- [[entities/anthropic-12-mcp-production-patterns]] — 替代/补充方案
- [[entities/agent-workflows]] — 替代/补充方案
- [[entities/livekit-agents-voice-ai-streaming-cascade-interruption-detection]] — 替代/补充方案


→ [[raw/articles/anthropic-12-mcp-production-patterns|原文存档]]

## 新增关联实体
- [[entities/livekit-agents-voice-ai-streaming-cascade-interruption-detection]]

## 2026-07-28 无状态化演进与网关视角（SUPP 2026-08-06）

> 来源：阿里云云原生《MCP 重回 HTTP 范式》（Higress 网关团队视角，2026-08）^[raw/articles/higress-mcp-stateless-http-paradigm-aliyun-2026-08-06.md]

### 核心改动：从有状态长连接回到无状态请求/响应

MCP 2026-07-28 版本把协议从有状态、依赖长连接的模型改回无状态的请求/响应模型（"重回 HTTP 范式"）——HTTP 无状态是 Web 架构最基础最成熟的做法。^[raw/articles/higress-mcp-stateless-http-paradigm-aliyun-2026-08-06.md]

- **退役握手与会话标识（SEP-2575、SEP-2567）**：过去依赖 initialize/initialized 握手 + Mcp-Session-Id 维持上下文，同一会话多次请求必须落到同一服务端实例（会话亲和性），负载均衡要么记住会话-实例绑定、要么实例间共享状态——横向扩容的额外架构成本。新版本每个请求自描述（协议版本/客户端身份/能力随请求携带），任何请求可落到普通轮询负载均衡后的任意实例，无需共享存储。^[raw/articles/higress-mcp-stateless-http-paradigm-aliyun-2026-08-06.md]
- **长连接交互替换为 MRTR**：elicitation、sampling、roots 等依赖服务端主动发起的交互被替换为多轮请求——服务端返回 input_required，客户端带上答案重试。^[raw/articles/higress-mcp-stateless-http-paradigm-aliyun-2026-08-06.md]
- **强制 Mcp-Method 和 Mcp-Name header（SEP-2243）**：网关、限流器可直接按 header 路由和计量，无需解析请求体——对网关/代理层是重要简化。^[raw/articles/higress-mcp-stateless-http-paradigm-aliyun-2026-08-06.md]
- **能力发现增强（SEP-2549）**：tools/list、prompts/list、resources/list 返回携带 ttlMs 和 cacheScope，客户端可缓存工具目录、重连后保持上游 prompt 缓存稳定；新增可选 server/discover 让能力发现更前置。^[raw/articles/higress-mcp-stateless-http-paradigm-aliyun-2026-08-06.md]

### 社区批判：最痛的槽点没被解决

无状态化解决的是部署和扩展问题（降低运维复杂度），但社区对 MCP 最集中的抱怨——**上下文拥挤和费钱**（开发者体验问题）——新版本几乎没有正面回应。^[raw/articles/higress-mcp-stateless-http-paradigm-aliyun-2026-08-06.md]

- **上下文拥挤根源**：工具定义前置加载，Agent 干活前要把所有可用工具说明读进上下文，工具一多光定义就占掉相当部分窗口
- **缓存是外围改善不是本质解法**：ttlMs/cacheScope 优化的是不要反复重新拉取工具清单，并没有降低单轮对话里工具定义占用的 token——该占的上下文仍然要占，只是重复获取次数减少。开发者真正想要的是**工具按需加载**（只在需要时把相关工具定义喂进上下文）
- **费钱是上下文拥挤的直接结果**：token 占用没下降，调用成本降不下来；新版本没有任何一条改动优化压缩工具
- **迁移成本**：无状态化是破坏性变更，依赖会话标识的实现需改造代码；弃用清单——Dynamic Client Registration 正式弃用（转向 CIMD）、Roots/Sampling/Logging 弃用、Legacy HTTP+SSE 传输进入退场倒计时。^[raw/articles/higress-mcp-stateless-http-paradigm-aliyun-2026-08-06.md]

### 架构视角结论

这次升级没有引入新颖机制——无状态、请求自描述、按 header 路由都是 Web 架构用了多年的老办法。MCP 绕一圈回到这些做法，是因为爆火后真正的考验从"是否定义了 Agent 连接外部系统的新标准"变成"能不能在规模化流量下保障调用方和维护方的体验"——后者需要扎实的架构设计和丰富的工程实践。^[raw/articles/higress-mcp-stateless-http-paradigm-aliyun-2026-08-06.md]

## 所属 MOC

- [[moc/ai-misc-topics-frontier|Ai Misc Topics Frontier]]
- [[moc/cybersecurity-privacy|Cybersecurity Privacy]]
- [[moc/layer-3-agent-engineering|Layer 3 Agent Engineering]]
