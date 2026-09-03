---
title: "100 行 vs 托管 harness 权衡"
created: 2026-06-12
updated: 2026-08-01
type: concept
tags: [concept, harness, tradeoff, claude-code, agentcore, build-vs-buy]
sources: [entities/claude-code-core-internals, entities/agentcore-harness, entities/agent-harness-architecture]
---

## 定义

100 行 vs 托管 harness 是 agent 工程的核心 build-vs-buy 决策。Boris Cherny 的 Claude Code 原型证明 100 行 Python 能跑通主循环；AWS AgentCore 证明完整托管能省运维。两条路并存——前者上限高定制深，后者地板高交付快。

## 核心范式

- **100 行原型路线**：完全控制、易调试、可深度定制；运维成本全自担
- **托管路线**：scale / 监控 / 安全合规即得；定制深度受限于平台 API
- **临界点**：< 5 个工程师选托管，> 20 工程师可承担自建
- **混合策略**：核心 loop 自建，observability / queue / storage 用托管

## 背景与提出

「100 行 Python 脚本就能搭一个 agent loop」——这是 Boris Cherny（Claude Code 创始人）在 2025 年的著名 demo。^[entities/agentcore-harness] 但这 100 行和 AgentCore 的企业级托管 harness 之间，存在一个根本性的工程权衡：自主性 vs 安全性。100 行脚本完全可控但没有护栏，托管 harness 护栏完善但定制受限。这个权衡不只是技术选择，是产品哲学选择。

## 范式细节

100 行 harness 的优势：完全透明（每一行你都能读懂）、完全可控（改任何行为只需要改代码）、零依赖（不绑定任何平台）、极低启动成本（5 分钟跑起来）。代价是：没有自动扩缩容、没有内置安全机制、没有可观测性、没有成本控制、没有错误恢复——全靠你自己写。托管 harness（AgentCore / Bedrock Agent / LangGraph Cloud）的优势反过来：安全合规开箱即用、自动扩缩容、内置可观测性、成本审计。代价是：定制受限于 API 接口、调试困难（黑箱运行时）、平台锁定、调试只能看 log 不能断点。^[entities/claude-code-core-internals]

## 局限与反对声音

权衡模型本身没有问题，但现实中的选择不是二元的。大多数项目从 100 行起步，长到需要托管时迁移——但迁移成本往往被低估（100 行到 1000 行是 10×，但 1000 行迁移到托管可能是 3× 重写）。另一个批评是：「100 行」是个虚假精确数——真正能跑的 agent loop 至少 300 行（含错误处理、重试、日志），而 300 行已经到了「自己维护开始痛苦」的临界点。真实决策点不是 100 行 vs 托管，而是「什么时候从自己写的 300 行迁移到框架」。

## 现实案例

Claude Code 本身走的就是从轻到重的路径：最早的 Boris demo 是 ~100 行 Python；到 2025 Q2 产品化时已经是一个完整的 harness（working set 管理、subagent 隔离、auto-compact）；到 2026 年加上 Review Mode 和安全 gate 后，复杂度已经远超任何个人能维护的脚本。Hermes Agent 走了另一条路：从一开始就设计为框架（skill 系统 + cronjob + MCP 客户端），但保留了一个极简的入口（CLI prompt）——用户可以选择用框架能力也可以只当 100 行 REPL 用。^[entities/hermes-agent]

## 现实案例

Boris Cherny 的 Claude Code 原型最初就是 100 行 Python——一个 while 循环调 Anthropic API，把结果输出给用户，加一个简单的 tool dispatch。^[entities/claude-code-core-internals] 这个原型在内部用了半年才演化成产品级的 Claude Code（现在是数万行 Python + Rust）。但「100 行原型 → 产品」的演化路径是 Claude Code 独有的优势——他们有内部产品化能力把 100 行变成 production。开源社区的对比案例是 LangChain：从未有过 100 行原型阶段，从一开始就堆功能，导致代码库在 2 年内膨胀到无法维护的程度。AWS AgentCore 走完全相反的路：从第一天就是企业级托管，没有 100 行原型的空间。^[entities/agentcore-harness] 真实决策点不是「100 行 vs 托管」，而是「我团队的工程能力能撑住 100 行到产品化的演化吗」——大多数团队的答案是不能，所以选托管。Hermes Agent 的选择是中间路径：核心 loop 写得很轻（类似 100 行），但 harness 周边功能（cron / skill / memory）通过插件化扩展，避免一次性堆到 1 万行。

## 实践启示

build-vs-buy 决策的真实算法是「团队规模 × 任务复杂度 × 时间预算」。团队规模 < 5 + 任务复杂度低 + 时间预算紧 → 选托管（AgentCore / Bedrock Agent），3 天接入 2 周上线。团队规模 5-15 + 任务复杂度中 + 时间预算中等 → 选混合（核心 loop 自建 + observability/queue 用托管），4 周建好 8 周上线。团队规模 > 20 + 任务复杂度高 + 时间预算宽松 → 选自建（参考 Claude Code 演化路径），3-6 月建好但天花板高。决策时一个常被忽略的因素是「退出成本」——选托管后想迁出，数据/状态/workflow 都需要重新映射，成本通常是新项目 30%。选自建后想迁入托管，重写成本可能高达 50%（因为自建代码假设了特定架构）。所以如果未来 1 年可能换平台，优先选更标准的接口（OpenAI API 兼容的），避免深度绑定某个平台 API。Hermes Agent 的选择是「自建 + 标准接口」：核心 harness 用 Hermes 自家实现，但所有工具调用走 OpenAI 兼容 API，方便未来切换底层 LLM provider。

## 与相邻概念的区分

和「自研 vs SaaS」传统决策的区别：传统 SaaS（数据库 / 消息队列）的接口标准，迁移成本主要是数据迁移；harness 的接口不标准，迁移成本包含整个 agent workflow 重写。和「微服务 vs monolith」的区别：微服务是部署粒度选择（每服务独立部署），harness 是运行时架构选择（整个 agent loop 在一个 runtime 里）。100 行 harness 本质上是 monolith 思维（一个进程一个 loop），托管 harness 是 serverless 思维（按调用计费）。和「开源 vs 商业」的区别：开源 harness（LangChain / LlamaIndex）代码可见可改，但运维责任自担；商业 harness（AgentCore / Bedrock Agent）运维责任转移但代码不可见。100 行脚本其实不在「开源 vs 商业」光谱上——它是「自己写一份」，比开源更彻底（自己拥有代码），比商业更灵活（自己定义接口）。这个特殊位置让 100 行原型成为「验证假设」的最佳工具——用最低成本测试「harness 应该长什么样」，验证后再决定选哪个托管平台。

## 临界点判断的细化

「5 个工程师」和「20 个工程师」的临界点其实是简化版。真实临界点是「团队中能维护 harness 的工程师数量 × 团队对 harness 定制深度的需求 × 团队对 harness 演进速度的容忍度」。三个维度的乘积决定选型。一个 3 人创业公司但需要深度定制（特定业务协议）应该选自建；一个 30 人大厂但需求标准化（通用客服 agent）应该选托管。一个常被忽略的因素是「合规要求」——金融/医疗/政务的合规要求决定 harness 必须有审计日志、权限控制、数据脱敏，这些在 100 行脚本里实现成本远高于托管平台（托管平台开箱即用）。所以「临界点」的真实算法包含合规因子：合规要求高 + 资源少 → 选托管；合规要求低 + 资源多 → 可选自建。Hermes Agent 的设计哲学是「substrate 提供 80% 的能力，团队写 20% 的定制」——这介于自建和托管之间，团队不需要维护全部 harness，只需要写 skill/plugin 扩展 harness。

## 在 wiki 中的关联

- [[entities/claude-code-core-internals|Claude Code 100 行原型]]
- [[entities/agentcore-harness|AgentCore 托管 harness]]
- [[concepts/harness-as-product-surface|harness 作为产品界面]]
- [[concepts/managed-agents-architecture|managed agents architecture]]
- [[concepts/production-agent-engineering|production agent engineering]]

## 进一步阅读

- [[entities/claude-code-core-internals]]
- [[entities/agentcore-harness]]
- [[entities/agent-harness-architecture]]

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
