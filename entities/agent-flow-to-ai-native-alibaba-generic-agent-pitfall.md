---

title: "从 Agent Flow 到 AI Native：通用 Agent 的架构反思"
created: 2026-08-30
updated: 2026-09-07
type: entity
tags: [agent, architecture, flow, ai-native, skill, orchestration]
sources: [raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall]
confidence: 0.75
---

# 从 Agent Flow 到 AI Native：通用 Agent 的架构反思

## 摘要

阿里技术团队陈以强的这篇一线研发反思，以自研的 Agent Flow 实践为起点，系统批判了"通用 Agent 万能论"：通用 Agent 把一切押注在 LLM 的通用能力上，却在真实用户需求面前既不可靠又不可复现。作者主张把任务链路拆成可闭环的"最小任务单元"，通过编排层把任务外包给合适的 Agent Node，用 Flow 甚至是 hardcode 承接真实需求，并从 AI Native 的本质（基于 LLM 真实解决用户问题）、研发提效的组织障碍（基建不 LLM-friendly）、组织扁平化的真实含义三个层面展开论证。^[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall.md]

## 核心要点

- **Agent Flow 的定义**：传统 Flow 的强化版——用户用自然语言描述目标，LLM 理解目标、生成 Flow、修改 Flow、运行 Flow；Flow 本身只是编排层，负责链路控制、任务下发与结果回收，具体任务由 Node 执行 ^[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall.md]
- **Node 可以是任何能力**：QwenWork、Codex CLI、影刀 RPA、千牛接口、内部运维 Agent——把 Agent 作为能力之一，而不是当成一切，跳出单个 Agent 上升到编排层和控制层 ^[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall.md]
- **最小任务单元理念**：大模型输出本质概率性且上下文窗口有限，每次派发的任务应是"有具体上下文、工作量适中、可闭环确认结果"的单元；YAML、Flow、hardcode 是可靠的，纯自然语言不可靠 ^[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall.md]
- **通用 Agent 的复现难题**：同一任务开两个对话可能走两条思路，一个成功一个失败；业界靠堆 Memory 解决，但至今没有通用 Agent 敢说记忆系统已彻底解决问题 ^[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall.md]
- **用户选择 Agent 的两个理由**：只有你能解决某些问题（独占数据/接口：聊天记录、商品数据、钉钉消息），以及你真的能解决问题（用户只关心"我的问题解决了吗"）^[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall.md]
- **Hardcode 是美丽的**：LLM 时代代码廉价，代码维护者变成 LLM 后，10 个 if else 比抽象工厂更容易理解；客户愿意付费的问题被解决，Agent 才算成功 ^[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall.md]
- **AI Native 的本质**：基于 LLM，让 AI 解决用户的问题——不是贴 AI 皮，不是让用户学 Loop/RAG/MCP，而是更轻松、更可靠、更便宜地解决问题并让用户愿意继续用、愿意付费 ^[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall.md]

## 深度分析

### 1．"通用 Agent 饮鸩止渴"：概率性、不可复现与记忆沼泽

作者对通用 Agent 的批判建立在三个递进的事实上：其一，LLM 输出本质概率性，同一任务两次对话可能产生两条完全不同的思路，一次成功一次失败——通用 Agent 无法向用户承诺"确定地、可靠地完成复杂任务"；其二，业界用 Memory 系统回答这个问题，但记忆系统的状态管理本身就是一个更深的沼泽，至今没有通用 Agent 敢宣称彻底解决；其三，用户要的不是大模型像程序员解 bug 那样"探索"路径，而是以最低认知负担获得确定的结果。这三条叠加起来，构成了对"world agent"式愿景的现实反驳：与其在通用能力的星辰大海里等待升级，不如先把用户眼前的具体问题用可靠的方式解决掉。^[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall.md]

### 2．编排层思维：把 Agent 降格为"能力之一"

Agent Flow 的架构立场是反"Agent 中心主义"的：Flow 只负责链路控制、任务下发和结果回收，具体的执行工作外包给合适的 Node；Node 可以是 QwenWork、Codex CLI、影刀 RPA、千牛接口甚至内部运维 Agent。这背后的工程逻辑是"最小任务单元（最小混沌单元）"——每个 Node 都是一个有具体上下文、工作量适中、可闭环确认结果的单元，让庞大复杂的任务变得受控。更重要的是，YAML/Flow/hardcode 的确定性远高于纯自然语言的不可靠性：基于 YAML 的 Flow 自生成与自维护，比松散的 Skill 更简单高效。这一设计把确定性还给编排层，把概率性限制在单个 Node 的边界内，与 Harness Engineering"确定性协调器 + 概率性模型"的职责切分思想一脉相承。^[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall.md]

### 3．AI Native 的鉴别标准：贴皮 vs 重构

作者给出了一个可操作的 AI Native 判别方法：不是看产品有没有用 LLM、架构是否先进，而是问"用户的一句自然语言到底能不能把任务完整跑通"。很多 AI 搜索把 LLM 别扭地接入推荐系统，是典型的 LLM 套皮；OneRec 这类直接用原生推荐大模型——虽然作者声明并非赞扬——才更接近真正的 LLM Native。判断标准落在使用结果上：更轻松、更可靠、更便宜、或者至少更自然，且用户愿意继续用、愿意付费。这个标准把评判权从架构师的品味转移到用户的实际支付意愿上，是对"技术名词泛滥"潮流的直接对冲。^[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall.md]

### 4．研发提效的真正瓶颈不是模型，是基建与组织

作者放弃自研 WarRoom 运维 Agent 的经历揭示了一个常被忽略的真相：阻碍 Agent 进入研发现场的不是模型不够聪明，而是过去 20 年的基建都是为"人类"设计的——日志系统需要人工授权且无法 A2A、代码库需要手动开权限、中间件没有开放 AK/SK 管理、外部模型无法访问内网。每一步都受阻，最终只能在"人肉胶水"与"人在回路"之间痛苦维持。而 Aone 团队把同样的想法做成了，恰恰因为位置好：运维 Agent 天然能拿到更多权限、更多基建、通过更多审核。这印证了"只有你能解决某些问题"——数据、接口与权限的独占性，才是 Agent 竞争力的真正来源。^[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall.md]

## 实践启示

1. **先问"用户的问题解决了吗"，再谈架构**：用户不在乎 RAG、MCP、Multi-Agent、Agent Loop；大部分真实需求是几个有顺序的 LLM Call 加两个 API。拒绝用复杂 Loop 烧 token 换客诉。
2. **把任务拆成可闭环的最小单元再外包**：每次派发给 LLM 的任务都应具备具体上下文、适中工作量、可确认结果；用 Flow/YAML 把链路确定性留在编排层，把概率性限制在节点内。
3. **不要羞于 hardcode**：LLM 时代代码廉价，"10 个 if else"比抽象工厂对 LLM 维护者更友好。客户愿意为"问题被解决"付费，而不是为抽象程度付费。
4. **用数据/接口的独占性建立护城河**：通用模型再有智能也拿不到你的数据库、聊天记录、商品数据——能够背靠核心产品和功能拿到独一无二数据的 Agent，才是用户离不开的 Agent。
5. **在权限与基建不友好的环境，先解决"LLM 友好化"**：日志 A2A、代码库权限、中间件 AK/SK、模型访问内网——这些基建改造比调优模型更能让 Agent 真正落地，也是超级程序员助手成立的前提。
6. **用"自然语言能否完整跑通任务"作为 AI Native 的验收标准**：砍掉所有挡路的东西，以一个用户问题的端到端闭环为设计单位，而不是以技术名词的齐全度为设计目标。

## 相关实体

- [[entities/agent-harness-production|Agent Harness 生产化]]
- [[entities/harness-engineering-core-patterns-claude-code|Harness Engineering 核心模式]]
- [[concepts/harness-engineering-framework|Harness Engineering 框架]]
- [[entities/on-device-harness-qwen38-27b-portable-computer|端侧模型专用 Harness]]

→ [[raw/articles/agent-flow-to-ai-native-alibaba-generic-agent-pitfall|原文存档]]