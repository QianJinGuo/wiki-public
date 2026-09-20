---

title: "如何为 Agent 设计产品？"
type: entity
created: 2026-07-04
updated: 2026-09-20
tags: [wechat, ai]
rating: v7c8
sources:
  - raw/articles/如何为-agent-设计产品
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 如何为 Agent 设计产品？

**来源**: 架构师（JiaGouX）｜**发布日期**: 2026-04-26｜**原文链接**: https://mp.weixin.qq.com/s/mlajGBnYpugyxjTDc7JGNA

---

## 摘要

本文把问题从"怎么给 coding agent 设计工具"推进到"当 Agent 成为产品的新调用方，该怎么为它设计产品"。作者以 Salesforce Headless 360 为观察对象，指出无头化的重点不是放弃 UI，而是把 UI 后面沉淀的平台能力翻出来，让 Agent 从更多入口调用。结论是：为 Agent 设计产品不等于把页面换成 API，而是把产品能力整理成 Agent 能理解、能调用、能被约束、能被审计的动作。^[raw/articles/如何为-agent-设计产品.md]

## 核心要点

- 调用链变了：过去是"用户 → UI → 平台"，现在是"用户 → 用户自己的 Agent → 产品自己的 Agent / 工具层 → 平台底座"。多了一个调用方，产品就不只需要给人看的界面，还需要给 Agent 看的说明书。^[raw/articles/如何为-agent-设计产品.md]
- Salesforce Headless 360（"No Browser Required"）把平台能力暴露成 API、MCP 工具或 CLI 命令，首批超过 100 个工具。
- 拆掉的是**入口垄断**，守住的是**运行底座**——最值钱的不是页面，而是客户数据、审批链路、字段口径、权限规则和"只有这家公司才这么干"的业务习惯。
- 真实的失败模式是"工具可用 ≠ Agent 会用"：Notion MCP 主动要求 Agent 先读自己的 Markdown 规范，格式几乎不出错；Slack MCP 让 Agent 按通用 Markdown 写，结果样式错乱。
- Agent 不会让软件被使用得更少，反而更频繁：Ramp 的 MCP 周活三个月翻 10 倍，用途已从"查一查"变成补字段、归类交易、附收据、跑审批。
- 进生产绕不开确定性：概率系统与确定性流程之间需要一个可版本化、可审计的中间层（如 Agent Script）。
- "Agent 友好的产品"可分五层栈：表面层、调用层、语义层、业务底座、治理层；语义层与治理层最容易被跳过。

## 深度分析

### 当 Agent 成为调用方：契约要面向模型而非人的直觉

人看界面会读按钮、读文案、自己补上下文，出错还会猜下一步；Agent 看到的是工具名、参数、返回值、错误消息和一组行为约束。同一个功能，对人可能只是一个按钮，对 Agent 最好是职责清楚、可调用、可恢复的动作。^[raw/articles/如何为-agent-设计产品.md]

Claude Code 团队的试验验证了这一点：他们试过让模型"好好提问"，最后最稳的版本是一个职责极其单一的 AskUserQuestion 工具——模型负责调用，界面负责渲染选项，成功点不在工具复杂，而在职责清楚。作者由此提出一个自检问题：把页面拿掉，只给一组工具和文档，Agent 能不能知道这件事怎么做、做到哪里算完成、错了怎么退回来？^[raw/articles/如何为-agent-设计产品.md]

### Headless 的三块拼图：把能力暴露成 API·MCP·CLI 三个调用面

Headless 360 的官方拆法是三层：能力接入（60+ MCP 工具与 30+ 预配置 skills，让外部 coding agent 直连企业组织）、表面分离（把"做什么"与"长什么样"解耦，一次定义渲染到 Slack、Teams、ChatGPT、移动端）、可信交付（Agent Script 开源 + Evals + A/B Testing + 可观测）。^[raw/articles/如何为-agent-设计产品.md]

这里有个容易忽略的风险：Headless 之后，平台会不会退化成"可被 Agent 查询的数据库"？若只暴露表、对象、CRUD，它很容易被上层 Agent 或数据中台绕过去。Agent 友好的平台必须回答两个层次的问题——不只是"这里有什么数据"，而是"在这个组织、这个角色、这个状态下，下一步能安全做什么"。底座和数据库差的正是这一句话。^[raw/articles/如何为-agent-设计产品.md]

### 工具职责单一与渐进式披露：成功率来自语义层而非模型能力

为了让 Agent 少猜，语义层必须显式化：工具名和描述能不能让它选对、参数 schema 是否明确、哪些字段可推断哪些必须显式给、失败能否恢复、返回值是给谁看的。很多时候问题不在 Agent 能力，而在产品给的语义太粗。^[raw/articles/如何为-agent-设计产品.md]

渐进式披露是配套手段：别把所有文档塞进系统提示词，先给索引，相关时再让 Agent 自己拉细节——它需要在正确的时刻拿到正确粒度的规则。面向 Agent 的"文档"也升级为一组并列产物：llms.txt 给文档索引与入口，OpenAPI 给接口形状，skill.md 把能力、必填参数与约束压成可直接阅读的摘要，MCP Server 把真实能力暴露成工具。文档由此从售后环节变成产品接口的一部分。^[raw/articles/如何为-agent-设计产品.md]

### 守住底座而非入口：无头化之后的产品边界与可组合性

Notion 与 Slack 的对照说明"工具可用 ≠ Agent 会用"：Notion 的 MCP 主动递上自己的 Markdown 规范，Claude 推送的内容几乎每次都格式正确；Slack MCP 放任 Agent 按通用 Markdown 写，消息要么样式怪、要么需要人工返工。传统 API 文档写给开发者，开发者会读、会理解、会写转换层；Agent 时代的调用多发生在运行时，系统得在需要的那一刻把规范、schema、policy 递过去。^[raw/articles/如何为-agent-设计产品.md]

因此 CLI 与 MCP 不必对立：CLI 更像动作入口，适合本地、快速、可组合，样板是少交互、输出可解析、失败有明确状态、危险动作支持 --dry-run、需要确认时用 --yes；MCP 更像治理入口，适合多团队、多租户、权限与审计更复杂的场景。产品团队不必押一个名词，先把能力整理到合适的入口里。^[raw/articles/如何为-agent-设计产品.md]

### 从 coding agent 到 product agent：迁移中的确定性缺口

Agent 进生产后问题变得非常具体：改一句提示词会不会影响原来跑通的流程？哪些步骤可以自由推理，哪些必须按业务规则执行？失败能不能回放？新旧版本能不能 A/B？最后是谁批准了这个动作？VentureBeat 报道里有个扎人的细节：早期 Agentforce 客户把 Agent 做进生产后反而不敢改了，因为系统太脆——动一点就不知道还稳不稳，测试全得重做。^[raw/articles/如何为-agent-设计产品.md]

Agent Script 的意义不是替代代码，而是承认自由推理与企业确定性流程之间需要一个中间层：一个可读、可版本化、可审计的 flat file，容纳 if/else、状态转移、变量、动作顺序以及子 Agent 与动作选择。这与 Harness 的思路一致——模型越强，外面的主循环、工具、上下文、权限、状态与回滚越重要。^[raw/articles/如何为-agent-设计产品.md]

## 实践启示

1. **先把高频能力拆成原子动作**：说清能做什么、前置条件、失败能否重试、成功有什么副作用。若先做成 CLI，至少要有 --help、--json、稳定 exit code、--dry-run 和非交互模式——这层是能力被 Agent 调用前最便宜的试验场。
2. **把权限与审计从 UI 下沉到平台**：安全只在前端页面，Agent 一接入就出事。调用要继承对象级/字段级安全、共享规则与 profile，动作可追踪。
3. **补语义层，别只补调用层**：先发 API、接 MCP 当然要做，但没有语义层 Agent 照样猜。工具描述讲边界，复杂规范做成可按需读取的资源。
4. **把反馈循环做进产品**：让工具调用带上 rationale，提供单独的 feedback 工具，设计捕捉上下文的参数；Agent 留下的是结构化失败路径，比人一句"不好用"精确得多。
5. **上线前能测试，上线后能观测**：离线测试、自定义评分、A/B Testing、调用链观测、失败回放、版本回滚，实际会变成企业 Agent 的基础设施。
6. **换一套指标与计费视角**：除了点击、漏斗、留存，还要看动作数量、流程完成率、自动化节省的时间与治理开销；当干活的是 Agent 而非人，按席位、按登录理解 SaaS 价值会越来越别扭。

## 相关实体

- [[concepts/harness-engineering-framework|Harness Engineering]] — 模型外面那套主循环、工具、上下文与验证，是本文论证的地基。
- [[concepts/harness-as-product-surface|Harness as Product Surface]] — 把 Harness 从内部工程手段读作对外产品界面。
- [[concepts/model-context-protocol-mcp|Model Context Protocol (MCP)]] — Headless 平台暴露能力的治理型入口。
- [[concepts/agent-as-software-3-0-substrate|Agent as Software 3.0 Substrate]] — "产品成为 Agent 运行底座"的更大图景。
- [[entities/salesforce-headless-software-losing-head-a16z|Salesforce Headless：软件失去头部]]
- [[entities/the-ui-is-dead-long-live-the-agent-servicenow-goes-headless|ServiceNow 也走向 Headless]]

---

→ [[raw/articles/如何为-agent-设计产品|原文存档]]
