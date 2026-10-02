---
title: "The Oracle and the Firm"
created: 2026-06-16
updated: 2026-10-02
type: entity
tags: [article, newsletter]
source_url: "https://calv.info/the-oracle-and-the-firm"
sources: [raw/articles/calv-oracle-and-the-firm]
review_value: 6
review_confidence: 5
review_recommendation: strong
review_stars: 4
score_validated: 2026-09-05
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# The Oracle and the Firm

> Source: [[raw/articles/calv-oracle-and-the-firm|原文存档]] ^[raw/articles/calv-oracle-and-the-firm.md]

## 摘要

calv.info 作者在对比使用 Fable 5（Anthropic 系）与 GPT-5.5（OpenAI 系）后指出：两大前沿实验室正沿着两条截然不同的训练路线演化 context management 能力。OpenAI/Codex 走 "Oracle" 路线——单一长线程 + 频繁 server-side compaction；Anthropic/Claude 走 "Firm"（公司）路线——把问题拆给大量 sub-agent 并行执行，只回传结论。文章用组织类比解释两条路线，并从成本、感知速度、"遗忘"（信息丢失）三个维度做了 tradeoff 分析，预言终局是两种路线的融合。^[raw/articles/calv-oracle-and-the-firm.md]

## 核心要点

- **Context management 是前沿模型的核心功课**：模型要在海量 token 上解题（tool call 探索 + 思考），最终产出结果；任务越难、运行越久，context management 的规模化越是关键。^[raw/articles/calv-oracle-and-the-firm.md]
- **Oracle 路线（OpenAI）**：Codex 保持单条长线程 + 高频 compaction，窗口虽小（~200k vs Opus 的 1m）却能长期保持连贯，细节不易丢失。^[raw/articles/calv-oracle-and-the-firm.md]
- **Firm 路线（Anthropic）**：Claude Code（Opus 4.1 起）积极派生 `Explore` 等 sub-agent，各自在独立 context window 内工作，只向父 agent 回传相关信息；Fable 5 更是把这一策略用到极致。^[raw/articles/calv-oracle-and-the-firm.md]
- **Server-side compaction 的两大红利**：实现可随时迭代而不影响客户端；API 层可把长线程路由到正确 GPU，获得更优的 K/V cache 命中。^[raw/articles/calv-oracle-and-the-firm.md]
- **成本差异**：Anthropic 的 sub-agent 缺乏主动沟通，常常重复搜索相同文件，token 开销更高。^[raw/articles/calv-oracle-and-the-firm.md]
- **感知速度差异**：Firm 路线 token 并行产出，看起来"干得更多"；Oracle 路线是串行推进。^[raw/articles/calv-oracle-and-the-firm.md]
- **"遗忘"机制差异**：sub-agent 判定不值得回传的事实会从 context 中彻底消失，导致 Anthropic 模型更容易"漏报"；compaction 也会丢信息，但丢的概率更低。^[raw/articles/calv-oracle-and-the-firm.md]
- **终局判断**：Anthropic 会改进（目前过于 lossy 的）compaction，OpenAI 会为 multi-agent 训练，两者终将合流。^[raw/articles/calv-oracle-and-the-firm.md]

## 深度分析

### Oracle vs Firm：一次组织架构类比

文章最精彩之处是把两种 context 管理策略映射到人类组织的两种形态。**Oracle** 像一位全知的独裁者或"先知"：所有与用户目标相关的信息都在同一条线程里流动，单点维护全局连贯性——与整体轨迹相关的小细节会被"记住"。**Firm** 则像一家真正的公司：每个成员（sub-agent）有自己的目标、输入与输出，只看到总信息量的一个子集，靠语言（message-passing）沟通，但永远看不到彼此脑子里的隐藏状态。^[raw/articles/calv-oracle-and-the-firm.md]

这个类比解释了观察到的行为差异：为什么 Claude Code 会"疯狂"地产出 sub-agent（Fable 5 尤甚），而 Codex 只在用户主动要求时才偶尔派生 sub-agent——前者是组织化的本能，后者是先知的例外。^[raw/articles/calv-oracle-and-the-firm.md]

### Compaction-heavy 串行 vs 并行 sub-agent 委派

Compaction 有两种朴素实现：一是让一个（通常更小的）模型基于轨迹输出一条新消息，例如把整段对话摘要压缩到 1k tokens；二是按类别删除调用，例如删掉所有 tool calls 后继续推理。还可以混合：压缩较早的消息、保留近期消息的全保真度。自 ChatGPT 5.3-Codex 起，OpenAI 把 compaction 原生内置到 Responses API 的服务端——当产出 token 逼近窗口上限时自动触发压缩，只保留相关信息。^[raw/articles/calv-oracle-and-the-firm.md]

Anthropic 并非不做 compaction，但其 compaction 更慢、且要求客户端持续升级，因此训练重心倒向了委派策略：把问题切成子问题，让每个 agent 在自己的 context window 内完成大量工作，只回传相关结论。这是两种"省 token"哲学的对立——压缩历史（串行、信息有损但全局可见）vs 分治问题（并行、局部充分但全局盲区）。^[raw/articles/calv-oracle-and-the-firm.md]

### K/V cache 经济学与 message-passing

Server-side compaction 不只是工程便利，更是经济学：因为压缩发生在 OpenAI 的服务端，API 可以针对长线程做精细化 K/V cache 调度，把线程路由到已经缓存了对应 K/V 状态的 GPU 上，大幅降低重复 prefill 成本。客户端侧实现 compaction（Anthropic 的路线）则拿不到这一层优化，这也是其训练 regime 更倚重 sub-agent 的合理推论。^[raw/articles/calv-oracle-and-the-firm.md]

在 Firm 路线中，message-passing 是唯一的信息通道：sub-agent 之间不共享 context，父子之间只传"报告"。这带来一个结构性风险——信息在接口处被过滤。是否回传某个事实，由 sub-agent 单点决定；一旦它判断某个事实"不值得报告"，该事实就从整个系统里消失了。这种有损通信是 Firm 路线的固有成本。^[raw/articles/calv-oracle-and-the-firm.md]

### Tradeoffs 与两种遗忘模式

三种 tradeoff 可以统一到"信息在何处被丢弃"这个视角：

| 维度 | Oracle（compaction） | Firm（sub-agent 委派） |
|---|---|---|
| 丢信息的环节 | 压缩时，模型基于完整轨迹做取舍 | 回传时，sub-agent 单点决定 |
| 遗忘概率 | 较低（"less work" 去省略或保留 token） | 较高（未回传即永久缺失） |
| 表现 | 长任务连贯、细节可回忆 | 更容易 misreport 或漏掉"明明研究过"的事实 |
| token 成本 | 串行、总量可控 | sub-agent 重复劳动、开销更大 |
| 感知速度 | 串行推进 | 并行产 token，观感更快 |

作者朋友的观察——Anthropic 模型偶尔"更不连贯、更容易误报事实"——正可以用 message-passing 解释：模型并非没做过研究，而是研究结果在回传边界处被丢弃了。这也是 compaction 与委派最本质的差别：**compaction 的遗忘是集中式、可审计的；委派的遗忘是分布式、不可见的。**^[raw/articles/calv-oracle-and-the-firm.md]

## 实践启示

1. **构建 agent harness 时先选遗忘模式**：长程任务优先考虑集中式 compaction（可审计、细节保留好）；探索面大、可并行的任务（codebase research）适合 sub-agent 分治——但要在回传协议里强制关键事实上报。
2. **Sub-agent 回传要有最小信息契约**：Firm 路线的信息丢失发生在接口处；给 sub-agent 定义"必须回传"的字段（找到的文件、否定性结论、阻塞点）能显著缓解 misreport。
3. **长线程服务的 K/V cache 是隐藏护城河**：server-side 状态管理 + GPU 亲和路由能同时改善延迟与成本，self-hosting agent 平台的团队值得在推理层复刻这一设计。
4. **别用"感知速度"评价 agent 好坏**：并行 sub-agent 产 token 多、观感快，但重复搜索可能让真实成本更高；串行 compaction 慢而省。评估应看单位任务成本与事实准确性。
5. **两条路线正在合流，架构应双向兼容**：设计 harness 时同时预留 compaction 钩子与 sub-agent 编排接口，避免押注单一范式。

## 相关实体

- [[entities/agent-harness-context-management-working-set|Agent Harness Context Management Working Set]]
- [[entities/claude-code-subagents-context-hygiene|Claude Code Subagents Context Hygiene]]
- [[entities/agent-context-compression-yexiaochai-11|Agent Context Compression]]
- [[entities/ai-agent-loops-claude-code-codex|AI Agent Loops: Claude Code & Codex]]
- [[entities/from-doer-to-director-the-ai-mindset-shift|From Doer to Director: The AI Mindset Shift]]
