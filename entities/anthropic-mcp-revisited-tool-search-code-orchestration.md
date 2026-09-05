---
description: Auto-generated placeholder
title: "Anthropic 最新博客：MCP 没死，它又来了"
created: 2026-04-30
date: 2026-05-19
source: "[[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration|原文存档]]"
type: entity
value: 7
tags: [claude-code, anthropic, agent, mcp, skill]
review_value: 8
sources: [raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration]
review_confidence: 9
updated: 2026-09-05
---

[[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration]] ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

# Anthropic 最新博客：MCP 没死，它又来了
> source: https://mp.weixin.qq.com/s/Sz2hzXiNCyf1YNzPbeUo5Q
> author: J0hn，AGI Hunt
> date: 2026-04-23
> tags: #MCP #Agent #Claude #Tool-Search #代码编排 #Cloudflare #Skills

## 核心摘要
Anthropic 发布官方博客《Building agents that reach production systems with MCP》，回应社区对 MCP 三大批评（贵、schema 臃肿、不可组合），给出具体解法。核心结论：本地开发 → CLI + Skills；云端生产 → MCP + Skills；MCP 并未过时，正在成为云端 Agent 的标准化接入层。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
--- ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

## 01 社区批评回顾
过去一个多月社区对 MCP 的批评集中在三点： ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| 问题 | 数据 | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
|------|------| ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| Token 成本高 | CLI 在 token 消耗上便宜 10~32 倍，1 万次操作 CLI ~$3.2 vs MCP ~$55.2（17 倍差距） | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| Schema 膨胀 | GitHub MCP 服务器 43 个工具定义，每次对话全塞上下文，单工具定义占 4,026 tokens | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| 上下文污染 | Perplexity CTO：72% 上下文窗口被 MCP 占掉 | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
--- ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

## 02 Anthropic 的回应：三条路各有地盘
Agent 连接外部系统有三条路： ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| 路径 | 适用场景 | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
|------|----------| ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| 直连 API | 简单、一对一场景，但 M×N 问题严重（10 Agent × 10 服务 = 100 个集成方案） | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| CLI | 本地和沙箱环境，天然自描述（--help）、可组合（jq/pipe），无上下文污染 | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| MCP | 云端生产 Agent（Claude Cowork / Managed Agents / 移动端 / Web 端），无本地文件系统 | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
**关键数据**：MCP SDK 月下载量从年初 1 亿 → 3 亿，用脚投票的人越来越多。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
--- ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

## 03 Token 解法
### 解法一：Tool Search（按需加载）
**传统方式**：43 个工具，55,000 tokens，全塞进上下文 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
**Tool Search**：Agent 先描述需求，系统运行时搜索匹配工具，只把相关几个拉进来 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

- 工具定义 token 消耗减少 **85% 以上**
- 工具选择准确率不下降
- **关键原则**：按意图分组工具，别按 API 分（意图级工具 vs 细粒度工具）
**效果**：GitHub MCP token 消耗：32 倍差距 → 约 7 倍差距（从 44,026 tokens 降到 ~10,000 tokens） ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

### 解法二：程序化工具调用
**核心思路**：别让模型当搬运工，让它写代码 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

- 工具返回结果不在直接丢回模型
- 在代码执行沙箱里处理：Agent 可循环/过滤/聚合
- 只把最终结果返回上下文
**效果**：复杂多步工作流减少约 **37% token 消耗** ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

### 综合效果
MCP vs CLI 成本：从 32 倍差距缩小到约 7 倍。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
--- ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

## 04 Cloudflare 的实践：代码编排模式
Cloudflare MCP 服务器覆盖约 **2,500 个 API 端点**。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
传统方式：2,500 个端点工具定义全塞上下文 → 不现实 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
Cloudflare 做法：只暴露 **2 个工具**： ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

- `search`：找到需要的 API
- `execute`：在服务端沙箱里执行脚本
整个工具定义只占约 **1K tokens**。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
**模式本质**：Agent 用 search 找到 API → 写一段简短脚本 → 通过 execute 在服务端沙箱跑 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
这个模式叫**「代码编排」**。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
**与 CLI + Skills 的异同**： ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

- 相同点：Skill 告诉 Agent「怎么干」，CLI 提供「用什么干」，Agent 写代码调用，中间数据不经过上下文
- MCP 版本：把 CLI 的哲学搬进 MCP 协议，跑在云端，走 MCP 协议而非本地命令行
**Anthropic 的真正意思**：好的 MCP 服务器应该像 CLI 一样设计。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
--- ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

## 05 Skills 转正
Anthropic 明确定义了两者关系： ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| 角色 | 职责 | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
|------|------| ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| MCP | 管「能力」——让 Agent 能连上 Snowflake、Databricks、BigQuery | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| Skills | 管「编排」——告诉 Agent 该怎么用这些连接完成具体任务 | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
**典型案例**：Claude 数据插件 = 10 个 Skills + 8 个 MCP servers，覆盖 Snowflake、Databricks、BigQuery、Hex 等 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
**新趋势**：Canva、Notion、Sentry 等第三方服务商已发布 MCP 服务器同时附带 Skills ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
**MCP 社区动态**：正在开发 Skills 直接从 MCP 服务器分发的扩展，API 升级时 Skills 自动更新版本 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
**Peter 在播客的观点**（被 Anthropic 间接认可）："MCP 推动了很多公司去做 API，这是好的。我现在可以看一个 MCP 然后把它做成 CLI。" ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
--- ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

## 06 MCP 的真正地盘
### 三问题的 Anthropic 回答
| 社区批评 | Anthropic 回答 | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
|----------|----------------| ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| schema 臃肿 | Tool Search 减少 85% | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| 不可组合 | 程序化调用，让 Agent 写代码处理 | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
| token 贵 | 代码编排模式（Cloudflare 2 工具覆盖 2500 端点） | ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

### 发展图景
``` ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
本地开发环境 → CLI + Skills（轻量、快速、上下文干净） ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
云端生产环境 → MCP + Skills（标准化、跨平台、认证完备） ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
简单场景     → 直连 API（别瞎折腾） ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
``` ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
**MCP 没有死**：它并非万能方案，但正在成为**云端 Agent 的标准化接入层**。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
--- ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

## 深度分析
1. **MCP正在重演SSH的成功路径**：SSH早期也被批评臃肿、复杂，最终凭借「标准化+安全+跨平台」成为服务器管理的王者。MCP的M×N问题催生了协议层的抽象需求，Tool Search本质上是把「连接器发现」从编译期搬到运行时，这是工程进化的必然而非妥协。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
2. **代码编排是Unix哲学的云端复兴**：Pipe、Jq、CLI在本地被验证了几十年，Cloudflare的search+execute模式本质上把这套哲学搬到了云端。关键洞察：Agent不需要知道工具的完整schema，只需要知道「做什么」，search机制承担了「能力发现」的角色——这是工具设计的范式转变。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
3. **Tool Search和程序化调用的Token下降可以叠加**：85%（工具定义压缩）+ 37%（工作流优化）不是独立事件，而是作用于token消耗的不同阶段——前者影响初始上下文建立成本，后者影响多轮对话成本。两者叠加意味着MCP在复杂任务上的相对劣势已大幅收窄。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
4. **「意图分组」原则颠覆了工具定义逻辑**：传统做法按API能力组织工具，Cloudflare案例证明应该按Agent意图分组——2个工具覆盖2500端点不是奇迹，而是把「怎么找到正确的API」这件事交给search机制，而不是让Agent在海量的细粒度工具中做选择题。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
5. **MCP vs CLI的分野本质是「数据位置」而非「场景简单/复杂」**：CLI适合有本地文件系统、数据不出墙的场景；MCP适合数据在远程、需要跨平台复用的场景。Skills解决「编排」问题，MCP解决「连接」问题，两者分工比竞争更准确。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

## 实践启示
1. **设计MCP服务器时，优先实现「搜索优先」接口**：参考Cloudflare的search+execute模式，将大量细粒度API包装为2-3个高层次意图工具（search/find + execute/run），即使你的服务有数百个端点，工具定义也能控制在1K tokens以内。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
2. **将工具按「意图」而非「API」重新分组**：在设计工具定义时，不要直接暴露内部API，而是先问「Agent想完成什么意图」，然后把对应的多个API调用封装为一个意图级工具。这样Tool Search才能真正发挥作用，否则搜索只能找到细碎的API而非有意义的工具组合。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
3. **对于涉及多工具的工作流，采用程序化调用而非直接反馈**：在代码沙箱中处理工具返回的中间结果（循环、过滤、聚合），只把最终结果返回给模型上下文。这不是MCP特有的优化，而是代码编排模式的核心——模型应该指挥代码执行，而不是自己当搬运工。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
4. **用「每任务总Token消耗」而非「每工具Token消耗」评估工具效率**：Tool Search减少了工具定义的token，但MCP的真正瓶颈是多轮对话中的中间结果反馈。评估时应该看完整任务的token消耗，才能看清楚代码编排模式是否真的带来了收益。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
5. **在本地开发和云端生产之间建立清晰的工具边界**：如果你的团队同时有本地CLI工具和云端MCP服务，应该让它们各自做自己最擅长的事——CLI用于快速迭代和敏感数据处理，MCP用于跨环境一致性和第三方集成。别试图用MCP替代本地开发工具，这是在用错误的工具做正确的事。 ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]
--- ^[raw/articles/anthropic-mcp-revisited-tool-search-code-orchestration.md]

## 相关链接
- Anthropic 博客原文：https://claude.com/blog/building-agents-that-reach-production-systems-with-mcp
## 相关实体
- [[entities/anthropic-mcp-revisited]]
- [[entities/anthropic-12-mcp-production-patterns]]
- [[entities/anthropic-官方生产级-agent-最佳实践12-个可复用的-mcp-设计模式]]
- [[entities/tencent-skill-writing-complete-playbook-jackjchou]]
- [[entities/anthropic-官方生产级-agent-最佳实践12-个可复用的-mcp-设计模式-v2]]

- [[entities/打造-ai-智能体专属的代码知识库gitnexus-完整上手攻略]]
- [[entities/cursor-recall-anthropic-daily-release-cat-wu]]
- [[concepts/wiki-audit-skill]]
- [[moc/tool-use-mcp-patterns|MOC]]