---
title: "Prompt Caching Engineering — Earendil Coding Agent Architecture"
created: 2026-07-28
updated: 2026-10-02
type: entity
tags: [agent, llm, prompt-caching, kv-cache, inference-optimization, coding-agent, earendil, pi, agent-architecture]
sources: [raw/articles/earendil-prompt-caching-coding-agents]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Prompt Caching Engineering — Coding Agent System Architecture

> **Background**：本文档基于 Earendil Engineering 发表的深度技术文章 "Prompt Caching In Agents"（2026-07-22），系统分析了 KV Cache 在编码 Agent 场景下的架构设计与工程权衡。Earendil 是 Pi（模块化 AI Agent 构建平台）的开发者。

## 核心矛盾：编码 Agent 的 Prompt 增长模式

编码 Agent 每轮交互发送的 prompt 大部分内容与上一轮相同——system prompt、tool definitions、对话历史、工具调用结果都重复传输。session 增长到数万到数十万 token 后，每轮完全重算 prefill 变得缓慢且昂贵 ^[raw/articles/earendil-prompt-caching-coding-agents.md]。

Prompt caching 使这变得经济，但极为脆弱：一个 tool definition 的变化、模型切换、provider 路由决策都可能将期望的"低成本增量请求"变成 full replay of context。缓存行为因此不只是实现细节或优化——它影响 latency、cost、tool design、session design，甚至 product features 的取舍。^[raw/articles/earendil-prompt-caching-coding-agents.md]

## KV Cache 基础设施两种模式

| 模式 | 机制 | 优势 | 劣势 |
|------|------|------|------|
| **Session Affinity** | KV cache 保持在计算的 GPU 上/附近，后续请求路由回同一 worker | 快速，无需跨网络搬运 KV blocks | 调度受限：worker 过载、重启、evict 都会导致缓存丢失 |
| **Distributed Cache** | KV blocks 存入另一层存储或跨 worker 可用 | 调度灵活，恢复能力强 | 移动、索引、保留 KV blocks 本身是系统难题 |

两种模式在实践中混合 GPU memory、host memory、local storage、remote storage、prefix-aware routing 和 eviction policy。^[raw/articles/earendil-prompt-caching-coding-agents.md]

## Tool Loadout 对缓存的破坏（关键洞察）

Tool definitions 通常出现在 prompt 的开头部分（被"折叠"进 system prompt）。添加/删除一个 tool、修改 schema、或改变序列化顺序都会将 prompt 的 first mismatch 点移到靠近开头的位置，使之后所有缓存失效。^[raw/articles/earendil-prompt-caching-coding-agents.md]

这是一个常见的陷阱：plugin 系统（MCP-style tool catalogs）延迟加载 tool 看似高效（少发 schema），但新扩展的 loadout 使缓存的多轮对话全部失效。**节省几千个 tool-schema token 可能导致数万个 conversation token 被重新处理。**

**Additive Tool Loading**：部分新模型 API 支持 tool 在 transcript 中的特定 tool result 位置可用，而非插入到原始 tool list。这保持旧 prefix 不变。Pi 对支持 native deferred-tool 机制的模型提供了此能力。^[raw/articles/earendil-prompt-caching-coding-agents.md]

## 缓存生存期与中断

默认缓存 TTL 短于正常编码活动周期。Anthropic 默认 5 分钟缓存是典型例子——跑测试 7 分钟、code review、午餐、会议后返回，缓存的 KV state 已经消失。用户眼中"持续活跃"的编码 session，在 provider 看来是"isolated requests"序列。^[raw/articles/earendil-prompt-caching-coding-agents.md]

## Pi 的缓存设计哲学：不激进修剪

Pi 不主动删除旧的 tool calls。删除中间内容会改变那个点的 prefix，存活对话可能需要重新处理。一次性重写的成本可能超过未来节省。^[raw/articles/earendil-prompt-caching-coding-agents.md]

Pi 偏好稳定、append-oriented 的 transcript。Compaction 仅当 context pressure 需要 lossy rewrite 时才使用，将其视为 cache reset 而非 cache failure。

## 缓存可见性

Pi 在交互 footer 显示累计 cache reads/writes（R/W），以及最新请求的 cache-hit rate（CH）。`/session` 命令提供完整视图：缓存 vs 非缓存 input、累计 hit rate、成本和估算的回收费（re-billed tokens）。支持配置 `showCacheMissNotices` 在 cache miss 时显示预警。^[raw/articles/earendil-prompt-caching-coding-agents.md]

## 缓存退化的常见原因

1. **Idling**：空闲时间超过 provider retention window
2. **Model/provider 切换**：KV state 是 model-specific 的
3. **Branch navigation**：/tree、rewind、fork 改变 active token 序列
4. **Compaction 或手动历史重写**：有意替换部分 prompt
5. **Tool 和 reasoning level 变更**：除非支持 additive loading 且是纯增量变化
6. **Dynamic system prompts**：timestamp、random values、extension prompt snippets
7. **Extension context transforms**：修改旧消息或 provider payload
8. **Provider routing and eviction**：prompt 完全一致但 KV blocks 在请求落地处不可用

## 深度分析

### Tool Loadout 缓存破坏的 token 经济学：省几千 schema token 赔数万 conversation token

工具定义位于 prompt 最前端，任何增删、schema 变更或序列化顺序变化都会把 first mismatch 点拉回到接近 prompt 开头，使后续全部缓存失效^[raw/articles/earendil-prompt-caching-coding-agents.md:41]。被节省的是缓存中最便宜的部分（warm read 的几千 schema token），被损坏的是最昂贵的部分（数万乃至数十万 conversation token 的 cold write + 全量 recompute）。对 plugin / MCP-style tool catalog 这类动态 loadout 系统，"按需加载"的直觉在缓存视角下往往是负收益——除非 provider 支持 additive tool loading（tool 在 transcript 中特定 tool result 位置生效而非改写原始 tool list），旧 prefix 得以保留^[raw/articles/earendil-prompt-caching-coding-agents.md:43]。

### Session Affinity vs Distributed Cache：调度权衡如何影响 provider 选型

Session affinity 把 KV cache 留在计算它的 GPU 附近，请求路由回同一 worker：命中时最快（无需跨网络搬运 KV blocks），但调度被锁死——worker 过载、重启、evict 都直接转化为缓存丢失^[raw/articles/earendil-prompt-caching-coding-agents.md:28]。Distributed cache 把 KV blocks 放入独立存储层、跨 worker 共享：调度灵活、恢复能力强，代价是搬运、索引、保留 KV blocks 本身的系统工程^[raw/articles/earendil-prompt-caching-coding-agents.md:30]。对 provider 选型，这个权衡直接映射为 SLA 语义：affinity 型方案下缓存命中是"尽力而为"——一次 worker 重启就够把数万 token 的 session 打回 full replay；distributed 型方案命中率下限更高、方差更小。实际系统往往混合两种模式与多级存储^[raw/articles/earendil-prompt-caching-coding-agents.md:31]。选 provider 应问的不是"支不支持 caching"，而是"缓存住在哪层、eviction 策略如何、路由是否 prefix-aware"——这三点决定长 session 的真实 token 成本曲线。

### 5 分钟 TTL 与人类工作节奏错配的成本模型

Anthropic 默认 5 分钟缓存 TTL 短于多数正常编码活动：跑一次测试套件、一次 code review、一顿午饭或一个会议都足以让 KV state 完全过期^[raw/articles/earendil-prompt-caching-coding-agents.md:47]。用户视角里"持续活跃"的编码 session，在 provider 计费视角里是一串 isolated requests——每个返回点都是一次冷启动 full prefill^[raw/articles/earendil-prompt-caching-coding-agents.md:47]。粗略建模：设中断间隔为 I、TTL 为 T，则 I < T 的回合享受 cache read 价格，I > T 的回合支付 full prefill。编码工作的真实中断分布（几十秒的编辑间隙与十几分钟的构建/会议混杂）意味着相当比例的回合落在 I > T 区间。这解释了为什么 Anthropic 同时暴露更长的 retention 控制选项^[raw/articles/earendil-prompt-caching-coding-agents.md:47]——为更长 TTL 多付的 write 溢价通常远低于反复 full replay 的损失。Agent 侧的对策是把可预测的长中断（长构建、跑测试）显式建模为缓存重置点，而非指望 TTL 撑过去。

### Append-oriented transcript vs 激进 compaction 的取舍

删除 transcript 中间内容会改变删除点之后的 prefix，其后所有存活的对话都可能需要重新处理；重写一次长缓存上下文的即时成本，可能超过移除少量本可廉价缓存 token 所节省的未来成本^[raw/articles/earendil-prompt-caching-coding-agents.md:51]。Pi 因此选择稳定、append-oriented 的 transcript：不主动删除旧 tool calls，把 compaction 保留给 context pressure 确实要求 lossy rewrite 的场景，且将其心理定位为 cache reset 而非 cache failure^[raw/articles/earendil-prompt-caching-coding-agents.md:53]。取舍的本质是记账方式：激进 compaction 的成本**确定且前置**（一次全量 reprocess），收益**不确定且后置**（未来每轮省下的少量 token，还要赌 session 会继续多久）；append-oriented 的成本**小额且持续**（多带旧内容），收益**确定且前置**（prefix 不动、缓存持续命中）。只有当 context pressure 本身构成硬约束（接近模型窗口上限）时，compaction 的确定性成本才被"不得不做"所正当化。对 prompt caching 敏感的 Agent 设计者，这里的一般原则是：**任何修改历史的功能（rewind、fork、手动编辑）都应被视为缓存失效事件来计价**。

## 实践启示

1. **把缓存命中率当作一等工程指标**：cache 行为影响 latency、cost、tool design、session design 甚至产品功能取舍^[raw/articles/earendil-prompt-caching-coding-agents.md:16]。Agent harness 应在 UI 暴露累计 cache R/W 与 hit rate（如 Pi 的 footer 与 `/session` 视图），让缓存退化从隐性问题变成可见事件。
2. **冻结 prompt 前缀**：tool definitions、system prompt、序列化顺序是缓存的单点故障源^[raw/articles/earendil-prompt-caching-coding-agents.md:41]。动态加载前先算账：省下的 schema token（缓存价）vs 损失的 conversation token（重算价），并优先 additive tool loading。
3. **消灭 dynamic system prompt**：timestamp、随机值、extension 注入的 prompt snippet 都是 first mismatch 的种子^[raw/articles/earendil-prompt-caching-coding-agents.md:61]。把可变内容移到 prompt 尾部或作为普通消息 append，而不是插在稳定前缀中间。
4. **按中断节奏规划 TTL 策略**：编码工作的中断分布（构建、测试、会议）大量超过 5 分钟默认 TTL^[raw/articles/earendil-prompt-caching-coding-agents.md:47]。能买长 retention 就买；不能的话，把可预测长中断建模为缓存重置点并接受对应冷启动成本。
5. **默认 append-only，compaction 是最后手段**：删除中间内容改 prefix、殃及全部后续缓存^[raw/articles/earendil-prompt-caching-coding-agents.md:51]。仅在 context pressure 构成硬约束时做 lossy rewrite，并把它当作 cache reset 显式计价，而非无声的"优化"。
6. **选 provider 时追问缓存架构而非缓存功能**：affinity 还是 distributed、KV blocks 存在哪一层、eviction 与路由策略是否 prefix-aware^[raw/articles/earendil-prompt-caching-coding-agents.md:28]^[raw/articles/earendil-prompt-caching-coding-agents.md:30]。同样的"支持 prompt caching"标签下，长 session 的真实成本可能相差数倍。

## 与已知实体关系

- [[entities/anthropic-prompt-caching-claude-code|Anthropic Prompt Caching (Claude Code)]] — Claude Code 的 prompt caching 工程实践，侧重 Anthropic 生态
- [[entities/anthropic_cache_tokenomics|Tokenomics of Claude's Cache]] — Anthropic 62.5 分钟缓存规则的 Token 经济学分析
- [[entities/amazon-bedrock-claude-prompt-cache-strategy|Bedrock Prompt Cache Strategy]] — AWS Bedrock 上的 Prompt Cache 策略设计
- [[entities/pi-mono|pi-mono — 模块化 AI Agent 构建平台]] — Earendil/Pi 的模块化 Agent 平台
- [[entities/openclacky-harness-prompt-cache|OpenClacky Harness Prompt Cache]] — OpenClacky 的 Harness Prompt Cache 实践
- [[entities/headroom-context-compression-cache-stabilization|Headroom Context Compression]] — Headroom 上下文压缩与缓存稳定性
